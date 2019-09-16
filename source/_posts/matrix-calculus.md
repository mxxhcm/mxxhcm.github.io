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

### 符号声明
小写字母$x,y$是标量，小写加粗字母$\mathbf{x},\mathbf{y}$是向量，大写加粗$\mathbf{X},\mathbf{Y}$是矩阵。标量和向量都可以看成是矩阵，将vector看成$1\times n$或者$m\times 1$的矩阵，将scalar看成$1\times 1$的矩阵。$\mathbf{X}^T $表示矩阵$\mathbf{X}$的转置，$tr(\mathbf{X})$表示迹，即对角线元素之和。$det(\mathbf{X}$或者$\vert \mathbf{X}\vert$表示行列式。

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

### 用途
Matrix calculus通常用于优化，常常用在拉格朗日乘子法中。包括Kalman滤波，高斯混合分布的EM算法，梯度下降法的的导数。

## 向量导数
向量可以看成只有一列的矩阵，最简单的矩阵导数应该是标量导数，然后是向量导数。这一节使用的是numerator layout（分子布局）。

### 向量对标量求导
### 标量对向量求导
### 向量对向量求导

## 矩阵导数
### 矩阵对标量求导
### 标量对矩阵求导

## Layout
事实上，有两种定义矩阵导数的方式，这两种方法刚好差一个转置。这也是我们平常矩阵求导最容易迷惑的地方。
求向量对向量的导数时，即$\frac{\partial \mathbf{y}}{\partial \mathbf{x}}, \partial \mathbf{x} \in \mathbb{R}^n , \partial \mathbf{y} \in \mathbb{R}^m $，有两种方式表示，一种结果是$m\times n$的矩阵，一种是$n\times m$的矩阵。这就有以下的layout：
1. Numetator layout，根据$\partial \mathbf{y}$和$\partial \mathbf{x}^T $进行布局。也叫Jacobian 公式，最后是一个$m\times n$的矩阵。
2. Denominator layout，根据$\partial \mathbf{y}^T $和$\partial \mathbf{x}$进行布局。也叫Hessian公式，最后是一个$n\times m$的矩阵，是numetator layout的转置。也有人把它叫做梯度，但是梯度通常指的是$\frac{\partial y}{\partial \mathbf{x}}$，即标量对向量求导，不需要考虑layout。

在计算梯度$\frac{\partial y}{\partial \mathbf{x}}$和$\frac{\partial\mathbf{y}}{\partial x}$的时候，也会有冲突。
1. 采用分子layout，$\frac{\partial y}{\partial \mathbf{x}}$是行向量和$\frac{\partial\mathbf{y}}{\partial x}$是列向量。
2. 采用分母layout，$\frac{\partial y}{\partial \mathbf{x}}$是列向量和$\frac{\partial\mathbf{y}}{\partial x}$是行向量。

最后计算标量对矩阵求导$\frac{\partial y}{\partial \mathbf{X}}$和矩阵对标量求导$\frac{\partial\mathbf{Y}}{\partial x}$。
1. 采用分子layout，$\frac{\partial y}{\partial \mathbf{X}}$和$\mathbf{Y}$的shape一样，$\frac{\partial\mathbf{Y}}{\partial x}$和$\mathbf{X}^T $的shape一样。
2. 采用分母layout，$\frac{\partial y}{\partial \mathbf{X}}$和$\mathbf{Y}$的shape一样，这里不用转置这是因为这样子好看。$\frac{\partial\mathbf{Y}}{\partial x}$和$\mathbf{X} $的shape一样。

接下来的公式主要对$\frac{\partial \mathbf{y}}{\partial x}, \frac{\partial y}{\partial \mathbf{x}}, \frac{\partial \mathbf{y}}{\partial \mathbf{x}}, \frac{\partial \mathbf{Y}}{\partial x},\frac{\partial y}{\partial \mathbf{X}}$这种组合分别考虑。
种类| 标量$y$|$m\times 1$列向量$\mathbf{y}$|$m\times n$矩阵$\mathbf{Y}$
:--:|:--:|:--:|:--:
标量$x$,numetator layout| $\frac{\partial{y}}{\partial{x}}$: 标量|$\frac{\partial{\mathbf{y}}}{\partial{x}}$: size为$m$的列向量|$\frac{\partial{\mathbf{Y}}}{\partial{x}}$: $m\times n$的矩阵
标量$x$,denominator layout| $\frac{\partial{y}}{\partial{x}}$: 标量|$\frac{\partial{\mathbf{y}}}{\partial{x}}$: size为$m$的行向量|$\frac{\partial{\mathbf{Y}}}{\partial{x}}$: $m\times n$的矩阵
$n\times 1$列向量$\mathbf{x}$,numetator layout|$\frac{\partial{y}}{\partial{x}}$: size为$n$的行向量|$\frac{\partial{\mathbf{y}}}{\partial{x}}$: $m\times n$的矩阵|
$n\times 1$列向量$\mathbf{x}$,denominator layout|$\frac{\partial{y}}{\partial{x}}$: size为$n$的列向量|$\frac{\partial{\mathbf{y}}}{\partial{x}}$: $n\times m$的矩阵|
$p\times q$矩阵$\mathbf{Y}$ numetator layout|$\frac{\partial{y}}{\partial{\mathbf{X}}}$: $q\times p$的矩阵| |
$p\times q$矩阵$\mathbf{Y}$ denominator layout|$\frac{\partial{y}}{\partial{\mathbf{X}}}$: $p\times q$的矩阵| |


### numerator layout
### denominator layout

## 公式

## 参考文献
1.https://en.wikipedia.org/wiki/Matrix_calculus
2.https://zhuanlan.zhihu.com/p/24709748
3.https://zhuanlan.zhihu.com/p/24863977

