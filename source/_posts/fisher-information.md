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

## Fisher information
当$\theta$是标量的时候。

### 最大似然估计
根据最大似然估计，有似然对数：
$$l = \log p(x|\theta)$$

## score function
根据似然对数，定义一个score function：
$$s(\theta) = \nabla_{\theta} \log p(x|\theta) $$
即score是似然对数的一阶导（梯度），似然对数是标量，score function是似然对数对$\theta$的导数。

### score function的期望
**定理** score function的期望是$0$
证明：
\begin{align\*}
\mathbb{E}\_{p(x|\theta)}\left[s(\theta)\right] & = \mathbb{E}\_{p(x|\theta)}\left[\nabla \log p(x|\theta)\right]\\\\
&=\int \nabla \log p(x|\theta) p(x|\theta) dx\\\\
&=\int \frac{\nabla p(x|\theta)}{p(x|\theta)} p(x|\theta) dx\\\\
&=\int \nabla p(x|\theta) dx\\\\
&=\nabla \int p(x|\theta) dx\\\\
&=\nabla 1\\\\
&= 0
\end{align\*}
即似然对数梯度向量的期望为是$0$。

### 第一种意义：score function的方差
用$I(\theta)$表示fisher information，它的定义就是score function（似然对数的一阶导）的方差：
$$I(\theta) = \mathbb{E} \left[ \left(\frac{\partial}{\partial \theta} \log f(\mathbf{X}; \theta) \right)^2 |\theta \right] = \int \left( \frac{\partial}{\partial \theta} \log f(\mathbf{X}; \theta) \right)^2 f(x;\theta)dx$$

随机变量的Fisher information总是大于等于$0$的，Fisher information不是某一个observation的函数。

### 第二种意义：
$$I(\theta) =  - \mathbb{E}\left[ \frac{\partial^2 }{\partial \theta^2 } \log f(\mathbf{X}; \theta) |\theta \right] $$
Fisher informaction可以看成似然对数的曲率，在最大似然的估计值附近，fisher信息大代表着图像陡而尖，fisher信息小代表着图像宽而平。

### 第三种意义：Cramer-Rao bound的不正式推导
Fisher informaction的导数是$\theta$无偏估计值方差的下界。换句话说，$\theta$的精确度被似然对数的fisher information限制了。

## Fisher information Matirx
当$\theta$是多维变量的时候。
### 第一种意义：协方差矩阵
$$I(\theta) = \mathbb{E}\left[s(\theta) s(\theta)^T\right]$$
根据协方差矩阵的定义：
$$\Sigma = \mathbb{E}\left[(X-\mathbb{E}(X))(X-\mathbb{E}(X))^T \right]$$
所以$s(\theta)$的协方差矩阵为：
$$\Sigma = \mathbb{E}\_{p(x|\theta)} \left[(s(\theta)-0)(s(\theta) - 0)^T \right] = \mathbb{E}\_{p(x|\theta)} \left[(s(\theta)s(\theta)^T \right] $$

### 第二种意义：Fisher information matrix和Hessian matrix
Fisher information matrix $F$等于似然对数的二阶导数（海塞矩阵），也是score function的一阶导，期望的负数。
$$I(\theta) =  - \mathbb{E}\left[ \frac{\partial^2 }{\partial \theta^2 } \log f(\mathbf{X}; \theta) |\theta \right] $$

证明：
\begin{align\*}
    \text{H}\_{\log p(x \vert \theta)} &= \text{J} \left( \nabla \log p(x \vert \theta) \right) \tag{\log p的二阶导等于\nabla \log p的雅克比矩阵}\\\\
    &= \text{J} \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \tag{log-derivative-trick}\\\\
    &= \frac{ \text{H}\_{p(x \vert \theta)} \, p(x \vert \theta) - \nabla p(x \vert \theta) \, \nabla p(x \vert \theta)^{\text{T}}}{p(x \vert \theta) \, p(x \vert \theta)} \tag{分数求导}\\\\
    &= \frac{\text{H}\_{p(x \vert \theta)} \, p(x \vert \theta)}{p(x \vert \theta) \, p(x \vert \theta)} - \frac{\nabla p(x \vert \theta) \, \nabla p(x \vert \theta)^{\text{T}}}{p(x \vert \theta) \, p(x \vert \theta)} \\\\
    &= \frac{\text{H}\_{p(x \vert \theta)}}{p(x \vert \theta)} - \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)}\right)^{\text{T}} \\\\
\end{align\*}
对上式取期望，得到：
\begin{align\*}
    \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \text{H}\_{\log p(x \vert \theta)} \right] &= \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \frac{\text{H}\_{p(x \vert \theta)}}{p(x \vert \theta)} - \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right)^{\text{T}} \right] \\\\
    &= \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \frac{\text{H}\_{p(x \vert \theta)}}{p(x \vert \theta)} \right] - \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)}\right)^{\text{T}} \right] \\\\
    &= \int \frac{\text{H}\_{p(x \vert \theta)}}{p(x \vert \theta)} p(x \vert \theta) \, \text{d}x \, - \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \nabla \log p(x \vert \theta) \, \nabla \log p(x \vert \theta)^{\text{T}} \right] \\\\
    &= \text{H}\_{\int p(x \vert \theta) \, \text{d}x} \, - \text{F} \\\\
    &= \text{H}\_{1} - \text{F} \\\\
    &= -\text{F} \\\\
\end{align\*}
最后得到：$\mathbf{F} = - \mathop{\mathbb{E}}\_{p(x \vert \theta)} \left[ \mathbf{H}\_{\log p(x|\theta)}\right] $


## 参考文献
1.https://en.wikipedia.org/wiki/Fisher_information
2.https://math.stackexchange.com/a/265933
3.https://www.zhihu.com/question/26561604/answer/33275982
4.https://wiseodd.github.io/techblog/2018/03/11/fisher-information/
