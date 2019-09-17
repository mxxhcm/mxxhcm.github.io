---
title: fisher information
date: 2019-09-16 10:24:34
tags:
 - 费雪信息
 - fisher information
 - 概率论
 - 最大似然估计
categories: 概率论
mathjax: true
---

## 先验知识
### 最大似然估计
根据最大似然估计，有似然对数：
$$l = log\ p(x|\theta)$$

## score function
根据似然对数，定义一个score function：
$$s(\theta) = \nabla_{\theta} log\ p(x|\theta) $$
即score是似然对数的一阶导（梯度），似然对数是标量，score function是似然对数对$\theta$的梯度向量。

## score function的期望
**定理** score function的期望是$0$
证明：
\begin{align\*}
\mathbb{E}\_{p(x|\theta)}\left[s(\theta)\right] & = \mathbb{E}\_{p(x|\theta)}\left[\nabla log\ p(x|\theta)\right]\\\\
&=\int \nabla log\ p(x|\theta) p(x|\theta) dx\\\\
&=\int \frac{\nabla p(x|\theta)}{p(x|\theta)} p(x|\theta) dx\\\\
&=\int \nabla p(x|\theta) dx\\\\
&=\nabla \int p(x|\theta) dx\\\\
&=\nabla \mathbf{1}\\\\
&= \mathbf{0}
\end{align\*}
即似然对数梯度向量的期望为$\mathbf{0}$向量。

### score function的协方差矩阵
根据协方差矩阵的定义：
$$\Sigma = \mathbb{E}\left[(X-\mathbb{E}(X))(X-\mathbb{E}(X))^T \right]$$
所以$s(\theta)$的协方差矩阵为：
$$\Sigma = \mathbb{E}_{p(x|\theta)} \left[(s(\theta)-0)(s(\theta) - 0)^T \right] = \mathbb{E}_{p(x|\theta)} \left[(s(\theta)s(\theta)^T \right] $$

## 第一种意义：score函数的方差
## Fisher information
用$I(\theta)$表示fisher information，它的定义就是score function的方差（协方差）：
$$I(\theta) = \mathbb{E}\left[s(\theta) s(\theta)^T\right]$$



## 第二种意义：对数似然二阶导数的期望的相反数
似然对数的海塞矩阵（二阶导数）期望的负数等于fish information matrix $F$。

## 参考文献
1.https://math.stackexchange.com/a/265933
2.https://www.zhihu.com/question/26561604/answer/33275982
3.https://wiseodd.github.io/techblog/2018/03/11/fisher-information/
