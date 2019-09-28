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

## Importance Sampling
Importance sampling是估计服从分布$p$的变量$x$的期望$f(x)$的一种方法。但是我们并不是从$p$中采样，而是从$q$中采样近似：
$$\mathbb{E}\_p\left[f(x)\right] = \mathbb{E}\_q\left[ \frac{p(x)f(x)}{q(x)}\right] \tag{1}$$
使用采样分布$q$估计分布$p$下的期望：
$$\mathbb{E}\_p\left[f(x)\right] \approx \frac{1}{n} \sum\_{i=1}^n \frac{p(x\_i)f(x\_i)}{q(x\_i)} x\_i\sim q\tag{2}$$
根据上面的公式，需要满足$p(x\_i)$不为$0$时，$q(x\_i)$也不为$0$。直接计算$\mathbb{E}\_p\left[f(x)\right]$和$\mathbb{E}\_q\left[f(x)\right]$，一般来说都是不同的，通过importance ratio调整权重，就可以使用$q$分布估计$p$分布的期望了。举个例子：
$$f(1) = 2, f(2) = 3, f(3) = 4, otherwise 0$$
概率分布$p$为：$p(x=1) = 0, p(x=2) = \frac{1}{3},p(x=3) = \frac{2}{3}$，概率分布$q$为：$q(x=1) = \frac{1}{3}, q(x=2) = \frac{1}{3}, q(x=3) = \frac{1}{3}$。计算期望，$\mathbb{E}\_p\left[f(x)\right] = \frac{11}{3}$，$\mathbb{E}\_q\left[f(x)\right] = 3$
使用importance ratio进行权重调整：
\begin{align\*}
\mathbb{E}\_p\left[f(x)\right] & = \mathbb{E}\_q\left[\frac{q(x)}{p(x)}f(x)\right] \\\\
& = \mathbb{E}\_q\left[\frac{p(x=1)}{q(x=1)}f(x=1) \right] + \mathbb{E}\_q\left[\frac{p(x=2)}{q(x=2)}f(x=2) \right] + \mathbb{E}\_q\left[\frac{p(x=3)}{q(x=3)}f(x=3) \right] \\\\
& = 0 + \frac{\frac{1}{3}}{\frac{1}{3}}*3 + \frac{\frac{2}{3}}{\frac{1}{3}}*4\\\\
& = \\\\
\end{align\*}
可以看出来，我们使用分布$q$估计除了分布$p$的期望。通过使用一个简单分布$q$进行采样，可以计算出$p$的期望。在RL中，通常通过复用old policy的sample trajectory学习current policy。

## Optimal Importance Sampling
Importance sampling使用$\mathbb{E}\_p\left[f(x)\right]\approx \frac{1}{N}\sum\_i \frac{p(x\_i)}{q(x\_i)}f(x\_i)$近似计算$\mathbb{E}\_p\left[f(x)\right]$。即使用采样近似估计，随着样本数量$N$的增加，期望值越准确。那么该如何减少方差呢，样本分布$q$应该满足：
$$q(x) \propto p(x)\vert f(x)\vert $$
直观上来说，这意味着为了减少方差，，我们需要采样return更大的点。

## Normalized importanct sampling
上面介绍的方法叫做unnormalized importance sampling。可以使用下里面的公式将unnormalized importance sampling转换为normalized importance sampling。
$$p(x) = \frac{\hat{p}(x)}{Z}$$
许多ML方法属于贝叶斯网络或者马尔科夫随机场，对于贝叶斯网络中，$p$很容易计算。但是当$p$是马尔科夫随机场时，$\sum\hat{p}(x)$是很难计算的。
\begin{align\*}
\mathbb{E}\_p\left[f(x)\right] & = \int f(x) p(x) dx\\\\
& = \int f(x) \frac{\hat{p}(x)}{Z} \frac{q(x)}{q(x)} dx\\\\
& = \frac{\int f(x) \hat{p}(x) \frac{q(x)}{q(x)}dx}{Z}\\\\
& = \frac{\int f(x) \hat{p}(x) \frac{q(x)}{q(x)} dx}{\int \hat{p}(x) dx}\\\\
& = \frac{\int f(x) \hat{p}(x) \frac{q(x)}{q(x)} dx}{\int \hat{p}(x)\frac{q(x)}{q(x)} dx}\\\\
& = \frac{\int f(x) q(x)\frac{\hat{p}(x)}{q(x)} dx}{\int q(x)\frac{\hat{p}(x)}{q(x)} dx}\\\\
& = \frac{\int f(x) r(x)q(x) dx}{\int r(x)q(x) dx}\qquad\qquad 记r(x) = \frac{\hat{p}(x)}{q(x)}\\\\
\end{align\*}
接下来用采样样本的求和近似积分求期望：
\begin{align\*}
\mathbb{E}\_p\left[f(x)\right] & = \frac{\int f(x) r(x)q(x) dx}{\int r(x)q(x) dx}\qquad\qquad 记r(x) = \frac{\hat{p}(x)}{q(x)}\\\\
& \approx \frac{\sum\_i f(x^i) r^i }{\sum r^i}\qquad\qquad 其中 r^i = \frac{\hat{p}(x^i ) }{q(x^i ) }\\\\
& = \sum\_i f(x^i) r^i  \frac{r^i}{\sum\_i r^i}\\\\
\end{align\*}
通过计算
这就避免了计算$Z$，这种方法叫做normalized importance sampling。但是需要付出一定代价，unnormalized importance sampling是无偏的，而normalized importance是有偏的但是方差更小。

## 参考文献
1.https://medium.com/@jonathan\_hui/rl-importance-sampling-ebfb28b4a8c6
