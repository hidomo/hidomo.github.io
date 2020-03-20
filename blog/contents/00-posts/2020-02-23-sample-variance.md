---
title: 样本方差 (Sample variance)
authors: [fancl20]
---

对于随机变量 `$X$` 的 N 个采样值 `$X_1..X_N$`，可得其均值与方差：

```
$$
\begin{aligned}
\bar X & = \frac{1}{N} \sum_{i=1}^{N} X_i \\
\operatorname{Var}(X) & = \frac{1}{N} \sum_{i=1}^{N} (X_i - \mu)^2
\end{aligned}
$$
```

若随机变量符合正态分布 `$\mathcal{N}(\mu, \sigma^2)$`，则可得：

```
$$
\begin{aligned}
\mathbb{E}[X] & = \mu \\
\mathbb{E}[\operatorname{Var}(X)] & = \sigma^2 \\
\end{aligned}
$$
```

可得均值的方差的期望为：

```
$$
\begin{aligned}
\mathbb{E}[\operatorname{Var}(\bar X)]
& = \mathbb{E}[\bar X^2] - \mathbb{E}[\bar X]^2 \\
& = \frac{1}{N^2} \mathbb{E}[(\sum_{i=1}^{N} X_i)^2] - \mu^2 \\
& = \frac{N \mathbb{E}[X_i^2] + \sum_{i=1,j=1,j \neq i}^{N}\mathbb{E}(X_iX_j)}{N^2} - \mu^2\\
& = \frac{N (\sigma^2 + \mu^2) + N(N-1) \mu^2}{N^2} - \mu^2 \\
& = \frac{\sigma^2}{N} \\
\end{aligned}
$$
```

若使用 `$\bar X$` 作为 `$ \mu $` 的近似计算方差，则：

```
$$
\begin{aligned}
\mathbb{E}\left[\frac{1}{N} \sum_{i=1}^{N} (X_i - \bar X)^2\right]
& = \mathbb{E}\left[\frac{1}{N} \sum_{i=1}^{N} (X_i - \mu + \mu - \bar X)^2\right] \\
& = \mathbb{E}\left[\frac{1}{N} \sum_{i=1}^{N} (X_i - \mu)^2\right] +
    \mathbb{E}\left[\frac{2}{N} \sum_{i=1}^{N} (X_i - \mu)(\mu - \bar X)\right] +
    \mathbb{E}\left[\frac{1}{N} \sum_{i=1}^{N} (\mu - \bar X)^2\right] \\
& = \mathbb{E}[\operatorname{Var}(X)] +
    \mathbb{E}[2(\bar X - \mu)(\mu - \bar X)] +
    \mathbb{E}[(\mu - \bar X)^2] \\
& = \mathbb{E}[\operatorname{Var}(X)] -
    \mathbb{E}[\operatorname{Var}(\bar X)] \\
& = \frac{N-1}{N} \sigma^2
\end{aligned}
$$
```

所以：

```
$$
\sigma^2
= \operatorname{Var}(X)
= \mathbb{E}\left[\frac{1}{N-1} \sum_{i=1}^{N} (X_i - \bar X)^2\right]
$$
```

换而言之，若希望从观测数据集 `$D$` 中求出最大似然的 `$\mu$` 与 `$\sigma^2$`，即最大化 `$p(D|\mu, \sigma^2)$`，则：

```
$$
p(D|\mu, \sigma^2) = \prod_{i=1}^{N}\mathcal{N}(X_i | \mu, \sigma^2)
$$
```

取对数，则：

```
$$
\begin{aligned}
\operatorname{ln}p(D|\mu, \sigma^2)
& = \sum_{i=1}^{N}\operatorname{ln}\mathcal{N}(X_i | \mu, \sigma^2) \\
& = \sum_{i=1}^{N}\operatorname{ln}\frac{1}{\sqrt{2\pi\sigma^2}}\operatorname{exp}\left\{-\frac{1}{2\sigma^2}(x-\mu)^2\right\} \\
& = \sum_{i=1}^{N}\operatorname{ln}\frac{1}{\sqrt{2\pi}} +
    \sum_{i=1}^{N}\operatorname{ln}\frac{1}{\sqrt{\sigma^2}} -
    \sum_{i=1}^{N}\frac{1}{2\sigma^2}(x-\mu)^2 \\
& = - \frac{N}{2}\operatorname{ln}(2\pi)
    - \frac{N}{2}\operatorname{ln}\sigma^2
    - \frac{1}{2\sigma^2}\sum_{i=1}^{N}(x-\mu)^2
\end{aligned}
$$
```

若要最大化，则对 `$\mu$` 求导可知 `$\mu_{ML} = \bar D$`，同样对 `$\sigma$` 求导可得 `$\sigma_{ML}^2 = \operatorname{Var}(D)$`。由此可知最大似然为 `$p(D|\bar X, \operatorname{Var}(D))$`。但是这种情况下存在对数据 `$D$` 的过拟合，或者称为有偏估计（biased estimator），既 `$\sigma_{ML}^2 = \frac{N-1}{N}\sigma^2$`（参见使用 `$\bar X$` 作为 `$ \mu $` 的近似计算方差的情况），所以可得：
```
$$
\mathcal{N}(X_i | \bar D, \frac{N}{N-1}\operatorname{Var}(D))
$$
```
为数据 D 的分布的最大似然估计