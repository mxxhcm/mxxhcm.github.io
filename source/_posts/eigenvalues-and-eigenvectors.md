---
title: eigenvalues and eigenvectors（特征值和特征向量）
date: 2019-08-28 17:21:43
tags:
 - 特征值
 - 特征向量
 - 线性代数
categories: 线性代数
mathjax: true
---

## 特征值和特征向量
这里介绍的东西都是针对于方阵来说的。

### 定义
$Ax=\lambda x $，满足该式子的$x$称为矩阵$A$的特征向量，相应的$\lambda$称为特征值。

### 求解
将$Ax=\lambda x$进行移项，得到$(A-\lambda I) x =0$，其中$A-\lambda I$必须是sigular（即不可逆），否则$x$必须为$0$向量。令$det (A-\lambda I)=0$，求出相应的$\lambda$和$x$。

### 属性
1. $n$个特征值的乘积等于行列式。
2. $n$个特征值之和等于对角线元素之和。

## 迹
### 定义
主对角线元素之和叫做迹（trace）。
$$\lambda_1 +\cdots + \lambda_n = trace = a_{11} + \cdots + a_{nn}$$

## 矩阵对角化
如果合适的使用矩阵$A$的特征向量，可以把$A$转换成一个对角矩阵。
假设$n\times n$的矩阵$A$有$n$个线性独立的特征向量$x_1,\cdots, x_n$，把它们当做列向量，构成一个新的矩阵$S=\left[x_1, \cdots, x_n\right]$。
$$AS = A\left[x_1, \cdots, x_n\right] = \left[\lambda_1 x_1, \cdots, \lambda_n x_n\right] = \left[x_1, \cdots, x_n\right] \begin{bmatrix} \lambda_1 &\cdots & 0 \\\\ \vdots&\lambda_i & \vdots\\\\ 0& \cdots & \lambda_n\end{bmatrix} = S\Lambda$$
即$AS = S\Lambda$，所以有$S^{-1} AS = \Lambda, A = S\Lambda S^{-1} $，这里我们假设$A$的$n$个特征向量都是线性无关的。$A, \Lambda$的特征值相同，特征向量不同。$A$的特征向量用来对角化$A$。

## 可逆和对角化
矩阵可逆和矩阵可对角化之间没有关联。
矩阵可逆和特征值是否为$0$有关，而矩阵可对角化与特征向量有关，是否有足够的线性无关的特征向量。

## 矩阵的幂
$A= S\Lambda S^{-1} $, 
$A^2 = S\Lambda S^{-1}S\Lambda S^{-1} = S\Lambda^2 S^{-1} $, 
$A^k = S\Lambda^k S^{-1}$
所以，$A^k $和$A$的特征向量相同，特征值是$\Lambda^k $。当$k\rightarrow \infty$时，如果所有的特征值$\lambda_i \lt 1$，那么$A^k \rightarrow 0$。 
## 微分方程

## 指数矩阵

## 对称矩阵
### 定义
满足$A= A^T $的矩阵$A$被称为对称矩阵。

$A = S\Lambda S^{-1} , A^T = {S^{-1}}^T \Lambda^T S^T$，而$A=A^T $，所以$S^{-1} = S^T $。

### 属性
1. 实对称矩阵只有实特征值
证明：假设$Ax= \lambda x$，我们能得到$A\bar{x} = \bar{\lambda} \bar{x}$，$\bar{x}$是$x$的共轭，对其转置，得到$\bar{x}^T A^T = \bar{x}^T A = \bar{x}^T \bar{\lambda}$。
在$Ax = \lambda x$的左边乘上$\bar{x}^T $得到$\bar{x}^T Ax = \bar{x}^T \lambda x$；在$\bar{x}^T A = \bar{x}^T \lambda$的右边同时乘上$x$得到$\bar{x}^T A x= \bar{x}^T \bar{\lambda} x$。
这两个式子左边一样，所以右边自然一样，即$\bar{x}^T \lambda x = \bar{x}^T \bar{\lambda} x$，而$\bar{x}^T x= \vert x\vert \gt 0 $，所以$\lambda = \bar{\lambda}$，即$\lambda$的虚部为$0$，即特征值都是实数。
2. 对称矩阵的特征向量可以是单位正交向量。
证明：假设$x,y$是矩阵$A$的两个特征向量，那么$Ax=\lambda_1 x, Ay = \lambda_2 y$。
$x^T \lambda_1^T y(\lambda_1 x)^T y = (Ax)^T y = x^T A^T y= x^T Ay= x^T\lambda_2 y$
因为$\lambda_1\neq \lambda_2$，所以$x^T y = 0$，即$x,y$正交。

### 谱定理（Spectral Theorem）
对称矩阵的对角化可以从$A=S\Lambda S^{-1} $变成$A=Q\Lambda Q^T$。
谱定理：每一个对称矩阵都有以下分解$A = Q\Lambda Q^T $，$\Lambda$是实特征值，$Q$是单位正交向量矩阵。
$$A = Q\Lambda Q^{-1} = Q\Lambda Q^T$$
$A$是对称的，$Q \Lambda Q^T $也是对称的。

## 正定矩阵
正定矩阵是对于对称矩阵来说的。

### 定义
如果对于所有的非零向量$x$，$x^T Ax$都是大于$0$的，我们称矩阵$A$是正定矩阵。

### 属性
1. 所有的$n$个主元都是正的
2. 所有的$n$个左上行列式都是正的
3. 所有的$n$个特征值都是正的
4. 对于任意$x\neq 0$，$x^T A x$大于$0$。
5. $A=R^T R$，$R$是一个具有$n$个独立column的矩阵。

如果任意矩阵$A$拥有以上属性中的任意一个，那么它就有其他四个性质。

### 半正定矩阵
如果对于所有的非零向量$x$，$x^T Ax$都是大于等于$0$的，我们称矩阵$A$是半正定矩阵。

### 属性
任何矩阵$A$都有$A^T A$和$A A^T $一定是半正定矩阵。

## 参考文献
1.MIT线性代数公开课
