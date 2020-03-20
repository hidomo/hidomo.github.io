---
title: Docker 魔法
authors: [fancl20]
---

### 使用 Docker 作为包管理器代替本地程序

类似官方 docker-compose 的[做法](https://docs.docker.com/compose/install/#install-as-a-container)，对于 cli 程序来说，交互主要通过 stdin/stdout/stderr/文件/网络，docker run 使用 -it 以后可以忽略 stdin/stdout/stderr 的差别，文件可以挂载全盘到容器内相同路径（对 Mac 来说可以挂载 /Users 目录，官方 docker for mac 也是类似的做法），网络的差异需要起代理来屏蔽，比较麻烦，不过场景不太多，可以先忽略。

这个做法有些地方需要 workaround

1. 由于可能使用相对路径，在 docker run 的时候需要指定路径 `-w $(pwd)`
2. stdin/stdout 可能不是 terminal，需要判断是否要加 `-t` 参数
3. 有些场景对程序速度有较高的要求（如 vscode 调用 clang-format 会超时），而 docker run 创建容器基本就要百毫秒，可以使用 docker exec 来避免创建容器的开销
4. 有的程序依赖环境变量，目前还没有太好的办法，只能将需要的变量在 docker run 的时候一个个透传进去

最后可以得到一个类似于如下的脚本

```bash
#!/bin/bash
if [ -t 0 ]; then
    interactive="-t"
fi

docker run --rm -i ${interactive} -v /Users:/Users -w $(pwd) --env TERM=xterm-256color \
    ${RUN_ARGS} ${IMAGE} "$@"
```

docker exec 的脚本更复杂一些，需要调用 docker run 的脚本，不过整体思路类似

```bash
#!/bin/bash
if [ -t 0 ]; then
    interactive="-t"
fi
if [ -z "$(docker ps | grep ${CONTAINER_NAME}$ 2> /dev/null)" ]; then
    export RUN_ARGS="${RUN_ARGS} -itd --name=${CONTAINER_NAME}"
    RUN_SCRIPT sh -c 'while true; do sleep 86400; done' &> /dev/null
fi
docker exec -i ${interactive} -w `pwd` \
    ${CONTAINER_NAME} "$@"
```

如需要 jq 就可以创建一个叫 jq 的脚本放在 PATH 中

```bash
#!/bin/bash
export CONTAINER_NAME=jq
RUN_SCRIPT jq "$@"
```

类似的，clang-format 的脚本

```bash
#!/bin/bash
export CONTAINER_NAME=clang
EXEC_SCRIPT clang-format "$@"
```

### 使用远程 Docker 代替本地程序

在使用 Docker 作为包管理器代替本地程序的基础上，考虑到笔记本性能不佳，用远程台式机配合 Docker 代替本地程序可以无缝提升本地程序的性能（如远程调用 cmake & make，可以大幅提升编译效率），或者无缝调用远程的 GPU。这个方案需要解决的问题主要是如何远程挂载本地磁盘到远程，docker 本身支持 nfs 可以达到这个效果。

首先需要在本地机器上起一个 nfsd（以 Mac 为例）

```bash
echo "/Users -alldirs -mapall=$(id -u):$(id -g) ${REMOTE_IP}" | sudo tee /etc/exports
sudo nfsd start # or restart
```

再在 docker 中添加对应 volumn

```bash
docker volume create --driver local \
    --opt type=nfs --opt o=addr=${LOCAL_IP},rw,nolock,hard,nointr,nfsvers=3 \
    --opt device=":/Users" ${VOLUME_NAME}
```

最后在 docker run 的参数中将 `-v /Users:/Users` 变为 `-v ${VOLUME_NAME}:/Users`

为了方便在拔掉 docker 之后自动切换成本机的 docker，在 PATH 中增加一个 docker 脚本

```bash
#!/bin/bash
if ifconfig | grep -q 192.168.0.1; then
    DOCKER_HOST=${REMOTE_DOCKER} /usr/local/bin/docker --config ${CONFIG} "$@"
else
    /usr/local/bin/docker "$@"
fi
```

这里依靠了一个静态 IP 来检测是不是插上了 dock，也可以使用更通用/安全的方法。

需要 workaround 的地方：

1. 由于我的台式机网线直接插在了笔记本 dock 上，本身没有外网，需要通过代理上网，docker build 等步骤需要配置代理，环境变量跨机器没有效果，根据[官方文档](https://docs.docker.com/network/proxy/)在**本地机器**的 docker 配置代理（httpProxy 与 httpsProxy）
2. 对于原本正常使用，需要 mount 本地路径的 docker run 来说，由于远程不存在对应的路径，需要改为直接访问被 mount 进来的路径，可以直接在远程机器的系统上 mount nfs 来避免这个问题，没有太好的在 docker 内解决的方法