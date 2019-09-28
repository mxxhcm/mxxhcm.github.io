---
title: reinforcement learning importance sampling
date: 2019-09-27 23:41:36
tags:
 - important sampling
 - 重要性采样
 - 强化学习
categories: 强化学习
mathjax: true
---

## importance sampling
Importance sampling是估计服从分布$p$的变量$x$的期望$f(x)$的一种方法。但是我们并不是从$p$中采样，而是从$q$中采样近似：
$$\mathbb{E}_p\left[f(x)\right] = \mathbb{E}_q\left[ \frac{p(x)f(x)}{q(x)}\right] \tag{1}$$
使用采样分布$q$估计分布$p$下的期望：
$$\mathbb{E}_p\left[f(x)\right] \approx \frac{1}{n} \sum_{i=1}^n \frac{p(x_i)f(x_i)}{q(x_i)} x_i\sim q\tag{2}$$
根据上面的公式，需要满足$p(x_i)$不为$0$时，$q(x_i)$也不为$0$。直接计算$\mathbb{E}_p\left[f(x)\right]$和$\mathbb{E}_q\left[f(x)\right]$，一般来说都是不同的，通过importance ratio调整权重，就可以使用$q$分布估计$p$分布的期望了。举个例子：
$$f(1) = 2, f(2) = 3, f(3) = 4, otherwise 0$$
概率分布$p$为：$p(x=1) = 0, p(x=2) = \frac{1}{3},p(x=3) = \frac{2}{3}$，概率分布$q$为：$q(x=1) = \frac{1}{3}, q(x=2) = \frac{1}{3}, q(x=3) = \frac{1}{3}$。计算期望，$\mathbb{E}_p\left[f(x)\right] = \frac{11}{3}$，$\mathbb{E}_q\left[f(x)\right] = 3$
使用importance ratio进行权重调整：
\begin{align\*}
\mathbb{E}_p\left[f(x)\right] & = \mathbb{E}_q\left[\frac{q(x)}{p(x)}f(x)\right] \\\\
& = \mathbb{E}_q\left[\frac{p(x=1)}{q(x=1)}f(x=1) \right] + \mathbb{E}_q\left[\frac{p(x=2)}{q(x=2)}f(x=2) \right] + \mathbb{E}_q\left[\frac{p(x=3)}{q(x=3)}f(x=3) \right] \\\\
& = 0 + \frac{\frac{1}{3}}{\frac{1}{3}}*3 + \frac{\frac{2}{3}}{\frac{1}{3}}*4\\\\
& = \\\\
\end{align\*}


## 参考文献
1.https://medium.com/@jonathan_hui/rl-importance-sampling-ebfb28b4a8c6
