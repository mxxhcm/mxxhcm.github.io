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
### 定义
如果合适的使用矩阵$A$的特征向量，可以把$A$转换成一个对角矩阵。
假设$n\times n$的矩阵$A$有$n$个线性独立的特征向量$x_1,\cdots, x_n$，把它们当做列向量，构成一个新的矩阵$S=\left[x_1, \cdots, x_n\right]$。
$$AS = A\left[x_1, \cdots, x_n\right] = \left[\lambda_1 x_1, \cdots, \lambda_n x_n\right] = \left[x_1, \cdots, x_n\right] \begin{bmatrix} \lambda_1 &\cdots & 0 \\\\ \vdots&\lambda_i & \vdots\\\\ 0& \cdots & \lambda_n\end{bmatrix} = S\Lambda$$
即$AS = S\Lambda$，所以有$S^{-1} AS = \Lambda, A = S\Lambda S^{-1} $，这里我们假设$A$的$n$个特征向量都是线性无关的。$A, \Lambda$的特征值相同，特征向量不同。$A$的特征向量用来对角化$A$。

### 属性
如果一个矩阵有$n$个不同的实特征值，那么它一定可对角化。
如果存在重复的特征值，可能但不一定可对角化，单位矩阵就有重复特征值，但可对角化。

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

### 属性
1. 实对称矩阵的特征值都是实数
证明：由$Ax= \lambda x$，得到$A\bar{x} = \bar{\lambda} \bar{x}$，$\bar{x}$是$x$的共轭，转置得：
$$\bar{x}^T A^T = \bar{x}^T A = \bar{x}^T \bar{\lambda}$$
$Ax = \lambda x$的左边乘上$\bar{x}^T $，在$\bar{x}^T A = \bar{x}^T \lambda$的右边同时乘上$x$：
$$\bar{x}^T Ax = \bar{x}^T \lambda x = \bar{x}^T A x= \bar{x}^T \bar{\lambda} x$$
即$\bar{x}^T \lambda x = \bar{x}^T \bar{\lambda} x$，而$\bar{x}^T x= \vert x\vert \ge 0 $，如果$x\neq 0$，则$\lambda = \bar{\lambda}$，即$\lambda$的虚部为$0$，即特征值都是实数。
2. 对称矩阵有单位正交的特征向量。
证明：假设$S = \left[v_1, \cdots, v_i, \cdots, v_n\right]$是矩阵$A$的特征向量矩阵，根据矩阵对角化公式：
$$A = S \Lambda S^{-1}  $$
而$A=A^T $，所以得到
$$S\Lambda S^{-1} = A = A^T = \left(S \Lambda S^{-1} \right)^T = S^{-T} \Lambda^T S^T = S^{-T} \Lambda S^T $$
可以得出$S^T = S^{-1} $，所以$S S^T = I$，即$v_i^T v_i = 1, v_i^T v_j = 0, \forall i\neq j$。
3. 所有的对称矩阵都是可对角化的。

### 谱定理（Spectral Theorem）
对称矩阵的对角化可以从$A=S\Lambda S^{-1} $变成$A=Q\Lambda Q^{-1} =Q\Lambda Q^T $。
谱定理：每一个对称矩阵都有以下分解$A = Q\Lambda Q^T $，$\Lambda$是实特征值，$Q$是单位正交向量矩阵。
$$A = Q\Lambda Q^{-1} = Q\Lambda Q^T $$
$A$是对称的，$Q \Lambda Q^T $也是对称的。

## 正定矩阵
正定矩阵，负定矩阵，半正定矩阵，半负定矩阵都是对于对称矩阵来说的。

### 定义
如果对于所有的非零向量$x$，$x^T Ax$都是大于$0$的，我们称矩阵$A$是正定矩阵。

### 属性
1. 所有的$n$个特征值都是正的
2. 所有的$n$个左上行列式都是正的
3. 所有的$n$个主元都是正的
4. 对于任意$x\neq 0$，$x^T A x$大于$0$。
5. $A=R^T R$，$R$是一个具有$n$个独立column的矩阵。

如果任意矩阵$A$拥有以上属性中的任意一个，那么它就有其他四个性质，或者说上面五个属性都可以用来判定矩阵是否为正定矩阵。

### 半正定矩阵
如果对于所有的非零向量$x$，$x^T Ax$都是大于等于$0$的，我们称矩阵$A$是半正定矩阵。

### 属性
对于任何矩阵$A$，$A^T A$和$A A^T $都是对称矩阵，并且它们一定是半正定矩阵。
假设$A = \begin{bmatrix} a&b\\\\c&d\end{bmatrix}$，如何判断$A^T A$是不是正定的？根据定义，判断$x^T (A^T A) x$的符号：
$$x^T (A^T A) x = x^T A^T Ax = (Ax)^T (Ax) = \vert Ax \vert $$
相当于计算向量$Ax$的模长，它一定是大于等于$0$的。
同理$A A^T $的二次型相当于计算$A^T x$的模长，大于等于$0$。

## 参考文献
1.MIT线性代数公开课
2.http://maecourses.ucsd.edu/~mdeolive/mae280a/lecture11.pdf
