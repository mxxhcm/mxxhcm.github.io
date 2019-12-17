---
title: fisher information
date: 2019-09-16 10:24:34
tags:
 - 概率论与统计
categories: 概率论与统计
mathjax: true
---

## score function
### 最大似然估计
根据最大似然估计，有似然对数：
$$l = \log p(x|\theta)$$

### score function
根据似然对数，定义一个score function：
$$s(\theta) = \nabla_{\theta} \log p(x|\theta) $$
即score是似然对数的一阶导（梯度），似然对数是标量，score function是似然对数对$\theta$的导数。


## Fisher information
当$\theta$是标量的时候。

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

### 第二种意义：参数真实值处似然对数二阶导期望的相反数
$$I(\theta) =  - \mathbb{E}\left[ \frac{\partial^2 }{\partial \theta^2 } \log f(\mathbf{X}; \theta) |\theta \right] $$
Fisher informaction可以看成似然对数对参数估计的能力，在最大似然的估计值附近，fisher信息大代表着图像陡而尖，参数估计能力好；fisher信息小代表着图像宽而平，参数估计能力差。

### 第三种意义：Cramer-Rao bound的不正式推导
Fisher informaction的导数是$\theta$无偏估计值方差的下界。换句话说，$\theta$的精确度被似然对数的fisher information限制了。

## Fisher information Matirx
当$\theta$是多维变量的时候。
### Preliminary
1.[雅克比矩阵和海塞矩阵](https://mxxhcm.github.io/2019/09/10/Jacobian-matrix-and-Hessian-matrix/)

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
    \text{H}\_{\log p(x \vert \theta)} &= \text{J} \left( \nabla \log p(x \vert \theta) \right) \\\\
    &= \text{J} \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \tag{log-derivative-trick}\\\\
    &= \frac{ \text{H}\_{p(x \vert \theta)} \, p(x \vert \theta) - \nabla p(x \vert \theta) \, \nabla p(x \vert \theta)^{\text{T}}}{p(x \vert \theta) \, p(x \vert \theta)} \tag{分数求导}\\\\
    &= \frac{\text{H}\_{p(x \vert \theta)} \, p(x \vert \theta)}{p(x \vert \theta) \, p(x \vert \theta)} - \frac{\nabla p(x \vert \theta) \, \nabla p(x \vert \theta)^{\text{T}}}{p(x \vert \theta) \, p(x \vert \theta)} \\\\
    &= \frac{\text{H}\_{p(x \vert \theta)}}{p(x \vert \theta)} - \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)} \right) \left( \frac{\nabla p(x \vert \theta)}{p(x \vert \theta)}\right)^{\text{T}} \\\\
\end{align\*}
第一个等号是$\log p$的海塞矩阵（二阶导）等于$\nabla \log p$（一阶导）的雅克比矩阵。
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


## Fisher information matrix和KL散度
两个分布$p(x|\theta)$和$p(x|\theta')$在$\theta'=\theta$处，KL散度的海塞矩阵等于fisher information matrix。
证明：
$$\text{KL}\left[p(x|\theta)||p(x|\theta')\right] = \int_x p(x|\theta)\log p(x|\theta)dx - \int_x p(x|\theta)\log p(x|\theta')dx$$
关于$\theta'$的一阶导为：
\begin{align\*}
\nabla_{\theta'} \text{KL}\left[p(x|\theta)||p(x|\theta')\right] & = \nabla_{\theta'}\int_x p(x|\theta)\log p(x|\theta)dx - \nabla_{\theta'}\int_x p(x|\theta)\log p(x|\theta')dx\\\\
& = - \int_x p(x|\theta) \nabla_{\theta'} \log p(x|\theta')dx\\\\
\end{align\*}
关于$\theta'$的二阶导为：
\begin{align\*}
\nabla_{\theta'}^2 \text{KL}\left[p(x|\theta)||p(x|\theta')\right] &=- \int_x p(x|\theta) \nabla_{\theta'} \log p(x|\theta')dx \\\\
&= - \int \nabla_{\theta'} p(x|\theta)\nabla_{\theta'} \log p(x|\theta') - \int p(x|\theta)\nabla_{\theta'}^2 \log p(x|\theta')  dx\\\\
&= - \int p(x|\theta)\nabla_{\theta'}^2 \log p(x|\theta') dx\\\\
\end{align\*}
当$\theta' = \theta$时：
\begin{align\*}
\text{H}\_{KL\left[ p(x|\theta)||p(x|\theta')\right]} & = - \int p(x|\theta)\nabla_{\theta'}^2 \log p(x|\theta')|\_{\theta'=\theta} dx\\\\
& = - \int p(x|\theta) H_{\log p(x|\theta)} dx\\\\
& = - \mathbb{E}\_{p(x|\theta)}\left[H_{\log p(x|\theta)} \right]\\\\
& = \text{F}\\\\
\end{align\*}


## 参考文献
1.https://en.wikipedia.org/wiki/Fisher_information
2.https://math.stackexchange.com/a/265933
3.https://www.zhihu.com/question/26561604/answer/33275982
4.https://wiseodd.github.io/techblog/2018/03/11/fisher-information/
5.https://wiseodd.github.io/techblog/2018/03/14/natural-gradient/
6.https://math.stackexchange.com/questions/2239040/show-that-fisher-information-matrix-is-the-second-order-gradient-of-kl-divergenc
