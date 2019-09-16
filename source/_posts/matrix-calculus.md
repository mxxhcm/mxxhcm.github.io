---
title: Matrix calculus
date: 2019-09-12 09:35:36
tags:
 - 矩阵求导
 - 矩阵微积分
categories: 线性代数
---

## 简介
矩阵微积分值得是使用矩阵或者向量表示因变量中每一个元素相对于自变量中每一个元素的导数。一般来说，自变量和因变量都可以是标量，向量和矩阵。

### 示例
考虑向量梯度，给定三个自变量，一个因变量的函数：$f(x_1, x_2, x_3)$，向量的梯度是：
$$\nabla f= \frac{\partial f}{\partial x_1}\mathbf{i} + \frac{\partial f}{\partial x_2}\mathbf{j} + \frac{\partial f}{\partial x_3}\mathbf{k}$$
其中$\mathbf{i,j,k}$表示坐标轴正方向上的单位向量。这类问题可以看成标量$f$对向量$x$求导数，结果依然是一个向量（梯度）：
$$\nabla f=\frac{\partial f^T }{\partial \mathbf{x}} = \left[\frac{\partial f}{\partial x_1} + \frac{\partial f}{\partial x_2} + \frac{\partial f}{\partial x_3}\right]^T $$

更复杂的情况是标量$f$对矩阵$\mathbf{X}$求导，叫做矩阵梯度(gradient matrix)。标量，向量，矩阵的组合总共有$9$中情况，其中六种情况可以用以下方式表示出来：
种类| 标量|向量|矩阵
:--:|:--:|:--:|:--:
标量| $\frac{\partial{y}}{\partial{x}}$|$\frac{\partial{\mathbf{y}}}{\partial{x}}$|$\frac{\partial{\mathbf{Y}}}{\partial{x}}$
向量|$\frac{\partial{y}}{\partial{x}}$|$\frac{\partial{\mathbf{y}}}{\partial{x}}$|
矩阵|$\frac{\partial{y}}{\partial{\mathbf{X}}}$| |

其中$x,y$是标量，$\mathbf{x},\mathbf{y}$是向量，$\mathbf{X},\mathbf{Y}$是矩阵。标量和向量都可以看成是矩阵，将vector看成$1\times n$或者$m\times 1$的矩阵，将scalar看成$1\times 1$的矩阵。

## 向量导数

## 矩阵导数

## Layout

## 公式

## 参考文献
1.https://en.wikipedia.org/wiki/Matrix_calculus
2.https://zhuanlan.zhihu.com/p/24709748
3.https://zhuanlan.zhihu.com/p/24863977

