---
title: orthogonality
date: 2019-08-27 19:21:44
tags:
 - 正交
 - orthogonality
 - 线性代数
categories: 线性代数
---

## Orthogonal vectors
给定向量$v,w$，如果$v^T w = 0$，那么这两个向量就是正交向量。

### Orthogonal subspaces
如果对于$\forall v\in V, \forall w\in W$，都有$v^tw = 0$，那么我们称subspaces $V,W$是orthogonal subspaces。

## Column space, nullsapce, row space, left nullspace的正交性
1. Row space和nullspace是正交的。
举个例子来证明吧：
$$A= \begin{bmatrix}c1&c2\end{bmatrix} = \begin{bmatrix}r1\\\\r2\end{bmatrix} = \begin{bmatrix}1&1&2&4\\\\0&0&1&3\end{bmatrix}$$
因为row spaces是row vector的linear combination，即$c_1 r_1+c_2 r_2$，而nullspace是$Ax=0$的所有解，即$x_1 c_1+x_2c_2  = 0$，这里的$0$是向量，可以推出来$r_1x = 0, r_2x =0 $，所以$c_1 r_1 x =0, c_2 r_2x = 0$，也就是说row space中的任意vector和nullspace中的vector都正交。  
直接使用数值方法证明：
用$x$表示$Ax=0$的$x$，用$A^Ty$表示row space，那么有
$$x^T(A^Ty) = (Ax)^T y = 0^Ty = 0$$

2. Column space和nullspace是正交的。

## Orthogonal complements
### 定义
如果一个subspace包含所有和subspace $V$正交的向量，称这个subspace是$V$的orthogonal complements（正交补）。

### 示例
Nullspace是row space的正交补（$\mathbb{R}^n$上）。
Left nullspace是column space的正交补（$\mathbb{R}^m$上）。


## Projections
如下图所示，左边是投影到一条直线上的结果，右边是投影到一个subspace上的结果

## $A^TA$是可逆的，当且仅当$A$有linear independent columns时
证明：
$A^TA$是一个$n\times n$的方阵，$A$的nullspace和$A^TA$的nullspace相等。
如果$Ax= 0$，那么$A^T Ax = 0$，所以$x$也在$A^TA$的nullspace中。如果$A^TAx=0$，那么我们要证明$Ax=0$，在左右两边同乘$x^T A^T Ax=0$，则$(AX)^T AX =0$，所以$\vert Ax\vert^2 =0$。也即是说如果$A^TAx=0$，那么$Ax$的长度为$0$，也就是$Ax=0$。
如果$A^TA$的columns是独立的，也就是说nullspace为空，所以$A$的columns也是独立的；同理，如果$A$的columns是独立的，那么$A^T$的columns也是独立的。

## Least Squares Approximations
$Ax=b$无解的情况，通常是等式个数大于未知数的个数，即$m\gt n$，$b$不在$A$的column space内。我们的目标是想让$e=b-Ax$为$0$，当这个目标不能实现的时候，可以在方程左右两边同时乘上$A^T$，求出一个近似的$\hat{x}$：
$$A^TAx = A^Tb$$
如何推导出这个结果，有以下几种方法：

### 最小化误差
1. 几何上
$Ax=b$的最好近似是$A\bar{x} = p$，最小的可能误差是$e=b-p$，$b$上的点的投影都在$p$上，而$p$在$A$的column space上，从直线拟合的角度上来看，$\bar{x}$给出了最好的结果。
2. 代数上
$b=p+e$，$e$在$A$的nullspace上，$Ax=b=p+e$我们解不出来，$A\bar{x} = p$我们可以解出来。

3. 积分

### 直线拟合

### 抛物线拟合

## Orthogonal Bases和Gram-Schmidi正交化


