---
title: 均值、方差、矩、协方差、独立和相关
date: 2019-08-08 19:01:15
tags:
 - 概率论
 - 均值
 - 方差
 - 样本均值
 - 样本方差
categories: 概率论
mathjax: true
---
## 均值（期望）
### 简单介绍
在概率论和统计学中，一个离散型随机变量的期望值是试验中每次可能的结果乘以相应概率的加和。
换句话说，期望值是变量取值的加权平均。期望并不一定包含于变量的值域内。

### 定义
如果$X$是概率空间$(\Omega, F,P)$中的随机变量，它的期望值$\mathbb{E}(X)$定义为：
$$\mathbb{E}(X) = \int_{\Omega} XdP$$
如果$X$是离散的随机变量，取值$x_1,x_2,\cdots$的概率分别为$p_1, p_2, \cdots, p_1+p_2+\cdots = 1$，它的期望定义为：
$$\mathbb{E}(X) = \sum_i x_ip_i$$

### 性质
- $\mathbb{E}(cX) = c\mathbb{E}(X)$
- $\mathbb{E}(X+c) = \mathbb{E}(X) + c$
- $\mathbb{E}(X+Y) = \mathbb{E}(X) + \mathbb{E}(Y)$
- $\mathbb{E}(XY) = \mathbb{E}(X)\mathbb{E}(Y)$，（$X,Y$独立）
- $\mathbb{E}(aX+bY) = a\mathbb{E}(X) + b\mathbb{E}(Y)$


## 方差
### 简单介绍
方差描述的是一个随机变量的离散程度。

### 定义 
假设$X$是服从分布$F$的随机变量，用$\mu=\mathbb{E}(X)$表示随机变量$X$的期望。
随机变量$X$的方差为：
$$Var(X) = \mathbb{E}\left[(X-\mu)^2\right]$$

### 性质
1. 方差总是大于等于$0$的
$$Var(X)\ge 0$$
2. 常数的方差为$0$
$$Var ( c ) = 0$$
3. 随机变量$X$加上一个常数$c$，它的方差不变
\begin{align\*}
Var(X+c) &= \mathbb{E}\left[(X+c - \mathbb{E}(X+c))^2\right]\\\\
&= \mathbb{E}\left[(X-\mathbb{E}(X))^2\right]\\\\
&=Var(X)
\end{align\*}
4. 随机变量$X$乘上一个常数$c$，它的方差变为原来的$c^2$倍。
\begin{align\*}
Var(cX) &= \mathbb{E}\left[(cX - \mathbb{E}(cX))^2\right]\\\\
&= \mathbb{E}\left[c^2 X^2 - 2c^2 X \mathbb{E}(X) + (\mathbb{E}(cX))^2\right]\\\\
& = c^2 \mathbb{E}\left[X^2 - 2X \mathbb{E}(X) + \mathbb{E}(X)\cdot \mathbb{E}(X)\right]\\\\
&=c^2 Var(x)
\end{align\*}


## 期望和方差关系
\begin{align\*}
Var(X) &= \mathbb{E}\left[(X-\mathbb{E}\left[X\right])^2\right]\\\\
&= \mathbb{E}\left[(X^2 - 2X \mathbb{E} \cdot \left[X\right]+\mathbb{E}\left[X\right] \cdot \mathbb{E}\left[X\right] )\right]\\\\
&= \mathbb{E}\left[(X^2) \right] - 2\mathbb{E}\left[(X \cdot \mathbb{E}\left[ X \right]) \right]+\mathbb{E}\left[(\mathbb{E}\left[X \right] \cdot \mathbb{E}\left[ X \right])\right]\\\\
&= \mathbb{E}    \left[(X^2) \right] -     \mathbb{E}\left[(\mathbb{E}\left[X \right] \cdot \mathbb{E}\left[ X \right])\right]\\\\
&= \mathbb{E}    \left[(X^2) \right] -  \mathbb{E}\left[X \right] \cdot \mathbb{E}\left[ X \right]\\\\
\end{align\*}

## 均值和样本均值
样本均值是均值的无偏估计。
假设用随机变量$X$表示所有男性的身高，得到一组样本值为$x_1, x_2, \cdots, x_n$，样本均值的计算方式如下：
$$\bar{X} = \frac{x_1 + \cdots +x_n}{n}$$
$\bar{X}$是对$\mu$的一个估计，真实的$\mu$我们不知道，但是我们能根据多个样本计算出来多个样本均值，这些样本均值的均值会收敛于均值。所以叫它无偏估计。

## 方差和样本方差
样本方差的分母如果是$n$的话，是方差的有偏估计，如果分母是$n-1$的话，是方差的无偏估计。
随机变量$X$方差的计算公式为：
$$Var(X) = \mathbb{E}\left[(X-\mu)^2\right]$$
但是我们往往不知道$X$的真实分布，所以经常用样本方差$S^2$来估计真实方差$Var(X)$：
$$S^2 = \frac{\sum_{i=1}^n (X_i - \mu)^2}{n}$$
而均值$\mu$往往也是不知道的，用样本均值$\bar{X}$代替均值$\mu$，得到
$$S^2 = \frac{1}{n-1}\sum_{i=1}^n (X_i - \bar{X})^2$$
为什么是$\frac{1}{n-1}$呢？
这里先给出几个辅助计算：
1. 样本均值的方差是样本方差的$\frac{1}{n}$
\begin{align\*}
Var(\bar{X}) &= Var(\frac{\sum_{i=1}^n X_i}{n})\\\\
& = \frac{1}{n^2} Var(\sum_{i=1}^n X_i)\\\\
& =  \frac{1}{n^2} \sum_{i=1}^n Var(X_i)\\\\
& = \frac{n\sigma^2 }{n^2 }\\\\ 
&=\frac{\sigma^2 }{n} 
\end{align\*}

如果使用$\frac{1}{n-1}\sum_{i=1}^n (X_i - \bar{X})^2$进行计算
\begin{align\*}
\mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n \left(X_i-\bar{X}\right)^2\right]&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n \left(X_i-\bar{X}+\mu-\mu\right)^2 \right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n \left(X_i-\mu - (\bar{X} - \mu)\right)^2 \right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n \left((X_i-\mu)^2 - 2(X_i-\mu)(\bar{X} - \mu) + (\bar{X} - \mu)^2 \right)\right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n (X_i-\mu)^2 - \frac{2}{n}\sum_{i=1}^n (X_i-\mu)(\bar{X} - \mu) + \frac{1}{n}\sum_{i=1}^n (\bar{X} - \mu)^2 \right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n (X_i-\mu)^2 - 2(\bar{X}-\mu)(\bar{X} - \mu) + \frac{1}{n}\sum_{i=1}^n (\bar{X} - \mu)^2 \right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n (X_i-\mu)^2 - 2(\bar{X}-\mu)^2 +  (\bar{X} - \mu)^2 \right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n (X_i-\mu)^2 - (\bar{X}-\mu)^2\right]\\\\
&= \mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n (X_i-\mu)^2 \right]-\mathbb{E}\left[ (\bar{X}-\mu)^2\right]\\\\
&= Var(X) - Var(\bar{X})\\\\
&= \sigma^2 - \frac{\sigma^2}{n}\\\\
&= \frac{n-1}{n}\sigma^2
\end{align\*}
最后得到$\mathbb{E}\left[\frac{1}{n}\sum_{i=1}^n \left(X_i-\bar{X}\right)^2  \right] = \frac{n-1}{n}\sigma^2 $，少算了$\frac{1}{n}\sigma^2 $，
所以，如果取$\mathbb{E}\left[S^2 \right] =\mathbb{E}\left[\frac{1}{n-1}\sum_{i=1}^n \left(X_i-\bar{X}\right)^2 \right]$，最后能得到$\mathbb{E}\left[S^2 \right] =\sigma^2$，是无偏估计。

## 矩
设$X$为随机变量，$c$为常数，$k$为正整数，称$\mathbb{E}\left[(X-c)^k\right] = \int_{-\infty}^{\infty} (X-c)^k f(X) dX $ 为$X$关于$c$的$k$阶矩。
1. $c=0, \alpha_k=\mathbb{E}\left[X^k\right] = \int_{-\infty}^{\infty} X^k f(X) dX  $称为$X$的$k$阶原点矩。
2. $c=\mathbb{E}\left[X\right],\mu_k=\mathbb{E}\left[(X-\mathbb{E}(X))^k \right] = \int_{-\infty}^{\infty}( X^k - \mathbb{E}(X))^k f(X)dX $称为$X$的$k$阶中心矩。

一阶原点矩为期望，任意随机变量的一阶中心距为$0$，二阶中心矩为方差。三阶中心矩表示偏度，任何对称分布的偏度为$0$，分布尾部在左侧比较长具有负偏度，分布尾部在右侧较长具有正偏度。四阶中心距表示峰度，俗称方差的方差。

## 协方差
方差和标准差通常是用来描述一维随机变量的特性。那么对于多维随机变量来说，怎么衡量它们之间的关系呢？引入了协方差衡量两维随机变量之间的关系。

### 定义
$\mathbb{E}\left[(X-\mathbb{E}(X))(Y-\mathbb{E}(Y)\right]$称为$X,Y$的协方差，记为$Cov(X,Y)$。
当$X=Y$时，其实就是方差的定义。

### 属性
1. \begin{align\*}
Cov(c_1X+c_2, c_3Y + c_4) &= \mathbb{E}\left[(c_1X +c_2 - \mathbb{E}(c_1X+c_2))(c_3Y +c_4 - \mathbb{E}(c_3Y+c_4))\right]\\\\
&= \mathbb{E}\left[(c_1X - c_1\mathbb{E}\left[X\right])(c_3Y - c_3\mathbb{E}\left[Y\right])\right]\\\\
&= c_1c_3\mathbb{E}\left[(X - \mathbb{E}\left[X\right])(Y - \mathbb{E}\left[Y\right])\right]\\\\
&= c_1c_3Cov(X,Y)\\\\
\end{align\*}
2. \begin{align\*}
Cov(X,Y) &= \mathbb{E}\left[(X-\mathbb{E}\left[X\right])(Y-\mathbb{E}\left[Y\right])\right]\\\\
&= \mathbb{E}\left[XY - \mathbb{E}\left[Y\right]X - \mathbb{E}\left[X\right]Y + \mathbb{E}\left[X\right]\mathbb{E}\left[Y\right]\right]\\\\
&= \mathbb{E}\left[XY\right] - \mathbb{E}\left[X\right]\mathbb{E}\left[Y\right]\\\\
\end{align\*}

### 独立与协方差之间的关系
若$X,Y$独立，则$Cov(X,Y) = 0$，因为独立变量有$\mathbb{E}(XY) = \mathbb{E}(X)\mathbb{E}(Y)$，所以：
$$\mathbb{E}(XY) - \mathbb{E}(X)\mathbb{E}(Y) = Cov(X,Y) = 0$$

### 协方差矩阵
用一个矩阵表示多维随机变量之间的关系，比如三个维度的随机变量。可以两两求出它们之间的协方差，因为协方差是对称的，所以这个矩阵是对称矩阵，对角线元素是方差。
协方差矩阵一般记为$\Sigma$，计算公式如下：
$$\sigma = \mathbb{E}\left[(\mathbf{X} - \mathbb{E}\left[\mathbf{X}\right])(\mathbf{X} - \mathbb{E}\left[\mathbf{X}\right])^T \right]$$
这里的$\mathbf{X}$是多维的随机向量，而方差和期望中的$\mathbf{X}$是一维的随机变量。



## 相关系数
### 定义
称$Cov(X,Y)/(\sigma_1,\sigma_2)$为$X,Y$的相关系数，记为$Corr(X,Y)$。

### 性质
1. 若$X,Y$独立，则$Covv(X,Y) = 0$
2. $\Vert Corr(X,Y)\Vert \le 1$，等号当且仅当$X$和$Y$有严格线性关系时取到
3. 当$Corr(X,Y)= 0$或者$Cov(X,Y) = 0$时，称$X,Y$不相关。

## 独立和相关
$X,Y$独立能够推出他们不相关，因为$Corr(X,Y) = 0$，满足不相关的定义。而$Corr(X,Y) = 0$不一定能推出$X,Y$独立。
即独立一定相关，但是相关不一定独立。有一个特例是正太分布，独立和相关互为充要条件。

## 参考文献
1.https://zh.wikipedia.org/wiki/%E6%9C%9F%E6%9C%9B%E5%80%BC
2.https://zh.wikipedia.org/wiki/%E6%96%B9%E5%B7%AE
3.https://www.zhihu.com/question/22983179/answer/404391738
4.https://www.matongxue.com/madocs/607.html
5.https://math.stackexchange.com/a/2113753/629287
6.https://blog.csdn.net/yangdashi888/article/details/52397990
