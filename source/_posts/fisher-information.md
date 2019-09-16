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
$$l = log p(x|\theta)$$

## score function
根据似然对数，定义一个score function：
$$s(\theta) = \nabla_{\thtea} logp(x|\theta) $$
即score是似然对数的一阶导，或者是梯度。

### score function的期望
**定理** score function的期望是$0$
证明：
\begin{align\*}
\mathbb{E}_{p(x|\theta)}\left[s(\theta)\right] & = \mathbb{E}_{p(x|\theta)}\left[\nabla logp(x|\theta)\right]\\\\
&=\int \nabla log p(x|\theta) p(x|\theta) dx\\\\
&=\int \frac{\nabla p(x|\theta)}{p(x|\theta)} p(x|\theta) dx\\\\
&=\int \nabla p(x|\theta) dx\\\\
&=\nabla \int p(x|\theta) dx\\\\
&=\nabla 1\\\\
&= 0
\end{align\*}

### score function的方差
根据方差的定义：
$$Var(X) = \mathbb{E}\left[(X-\mathbb{E}(X))^2\right]$$
所以：
$$Var(s(\theta)) = \mathbb{E}_{p(x|\theta)} \left[(X-\mathbb{E}(X))^2\right]$$


## 参考文献
1.https://math.stackexchange.com/a/265933
2.https://www.zhihu.com/question/26561604/answer/33275982
3.https://wiseodd.github.io/techblog/2018/03/11/fisher-information/
