---
title: linear algebra matrix calculus
date: 2019-09-12 09:35:36
tags:
 - 线性代数
categories: 线性代数
mathjax: true
---

## 符号声明
小写字母$x,y$是标量，小写加粗字母$\mathbf{x},\mathbf{y}$是向量，大写加粗$\mathbf{X},\mathbf{Y}$是矩阵。标量和向量都可以看成是矩阵，将vector看成$1\times n$或者$m\times 1$的矩阵，将scalar看成$1\times 1$的矩阵。$\mathbf{X}^T $表示矩阵$\mathbf{X}$的转置，$tr(\mathbf{X})$表示迹，即对角线元素之和。$det(\mathbf{X})$或者$\vert \mathbf{X}\vert$表示行列式。

## 基础
### 迹
$$ tr(\mathbf{A}) = tr(\mathbf{A}^T )$$
$$ tr(\mathbf{A}\mathbf{B}) = tr(\mathbf{B}\mathbf{A})$$
$$ tr(\mathbf{A}+\mathbf{B}) = tr(\mathbf{B})+tr(\mathbf{A})$$

### 行列式
...

## 简介
矩阵微积分值得是使用矩阵或者向量表示因变量中每一个元素相对于自变量中每一个元素的导数。一般来说，自变量和因变量都可以是标量，向量和矩阵。


### 示例
考虑向量梯度，给定三个自变量，一个因变量的函数：$f(x_1, x_2, x_3)$，向量的梯度是：
$$\nabla f= \frac{\partial f}{\partial x_1}\mathbf{i} + \frac{\partial f}{\partial x_2}\mathbf{j} + \frac{\partial f}{\partial x_3}\mathbf{k}$$
其中$\mathbf{i,j,k}$表示坐标轴正方向上的单位向量。这类问题可以看成标量$f$对向量$x$求导数，结果依然是一个向量（梯度）：
$$\nabla f=(\frac{\partial f }{\partial \mathbf{x}})^T = \left[\frac{\partial f}{\partial x_1} + \frac{\partial f}{\partial x_2} + \frac{\partial f}{\partial x_3}\right]^T $$

更复杂的情况是标量$f$对矩阵$\mathbf{X}$求导，叫做矩阵梯度(gradient matrix)。标量，向量，矩阵的组合总共有$9$中情况，其中六种情况可以用以下方式表示出来：
种类|标量|向量|矩阵
:--:|:--:|:--:|:--:
标量 | $\frac{\partial{y}}{\partial{x}}$ | $\frac{\partial{\mathbf{y}}}{\partial{x}}$ | $\frac{\partial{\mathbf{Y}}}{\partial{x}}$
向量 | $\frac{\partial{y}}{\partial{x}}$ | $\frac{\partial{\mathbf{y}}}{\partial{x}}$ | 
矩阵 | $\frac{\partial{y}}{\partial{\mathbf{X}}}$ | | 

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
标量对向量求导：
$\frac{\partial y}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y}{\partial x_1} & \cdots &\frac{\partial y}{\partial x_n}\end{bmatrix}$
向量对标量求导：
$\frac{\partial \mathbf{y}}{\partial x} = \begin{bmatrix}\frac{\partial y_1}{\partial x} \\\\ \vdots \\\\ \frac{\partial y_m}{\partial x}\end{bmatrix}$
向量对向量求导：
$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y_1}{\partial x_1} & \cdots & \frac{\partial y_1}{\partial x_n}  \\\\ \vdots \\\\ \frac{\partial y_m}{\partial x_1} & \cdots & \frac{\partial y_m}{\partial x_n}\end{bmatrix}$
标量对矩阵求导：
$\frac{\partial y}{\partial \mathbf{X}} = \begin{bmatrix}\frac{\partial y}{\partial x_{11}} & \cdots & \frac{\partial y}{\partial x_{p1}}  \\\\ \vdots \\\\ \frac{\partial y}{\partial x_{1q}} & \cdots &\frac{\partial y}{\partial x_{pq}}\end{bmatrix}$

下列公式只有numerator layout，没有denominator-layout：
矩阵对标量求导：
$\frac{\partial \mathbf{Y}}{\partial x} = \begin{bmatrix}\frac{\partial y_{11}}{\partial x}&\cdots \frac{\partial y_{1n}}{\partial x}  \\\\ \vdots \\\\ \frac{\partial y_{m1}}{\partial x} & \cdots & \frac{\partial y_{mn}}{\partial x}\end{bmatrix}$
矩阵微分：
$dx = \begin{bmatrix}dx_{11} & \cdots & dx_{1n} \\\\ \vdots \\\\ dx_{m1} & \cdots & dx_{mn}\end{bmatrix}$

### denominator layout
标量对向量求导：
$\frac{\partial y}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y}{\partial x_1} & \cdots &\frac{\partial y}{\partial x_n}\end{bmatrix}$
$\frac{\partial y}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y}{\partial x_1} \\\\ \cdots \\\\ \frac{\partial y}{\partial x_n}\end{bmatrix}$
向量对标量求导：
$\frac{\partial \mathbf{y}}{\partial x} = \begin{bmatrix}\frac{\partial y_1}{\partial x} & \vdots & \frac{\partial y_m}{\partial x}\end{bmatrix}$
向量对向量求导：
$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \begin{bmatrix}\frac{\partial y_1}{\partial x_1} & \cdots & \frac{\partial y_m}{\partial x_1}  \\\\ \vdots \\\\ \frac{\partial y_1}{\partial x_n} & \cdots & \frac{\partial y_m}{\partial x_n}\end{bmatrix}$
标量对矩阵求导：
$\frac{\partial y}{\partial \mathbf{X}} = \begin{bmatrix}\frac{\partial y}{\partial x_{11}} &\cdots & \frac{\partial y}{\partial x_{1q}}  \\\\ \vdots \\\\ \frac{\partial y}{\partial x_{p1}} & \cdots & \frac{\partial y}{\partial x_{pq}}\end{bmatrix}$

## 公式
### 向量对向量求导公式
### 标量对向量求导公式
### 向量对标量求导公式
### 标量对矩阵求导公式
### 矩阵对标量求导公式
### 标量对标量求导公式

### 微分形式的公式
通常来说使用微分形式然后转换成导数更简单。但是只有在使用numerator layout才起作用。

表达式|结果(numerator layout)
:-:|:-:
$d(\mathbf{A})  $ | $0$
$d(a\mathbf{X})$ | $ad\mathbf{A}$
$d(\mathbf{X}+\mathbf{Y})$| $d\mathbf{X}+d\mathbf{Y}$
$d(tr(\mathbf{X}))$ | $tr(d\mathbf{X})$
$d(\mathbf{X}\mathbf{Y})$| $(d\mathbf{X})\mathbf{Y}+\mathbf{X}(d\mathbf{Y})$
$d(\mathbf{X}^{-1} ) $| $- \mathbf{X}^{-1} (d\mathbf{X}) \mathbf{X}^{-1} $
$d(\vert\mathbf{X} \vert)$ | $\vert\mathbf{X}\vert tr(\mathbf{X}^{-1} d\mathbf{X}) = tr(adj(\mathbf{X})d\mathbf{X})$
$d(ln \vert\mathbf{X} \vert)$ | $tr(\mathbf{X}^{-1} d\mathbf{X})$
$d(\mathbf{X}^T) $| $(d\mathbf{X})^T $
$d(\mathbf{X}^H ) $| $(d\mathbf{X})^T $

其中$\mathbf{A}$不是$\mathbf{X}$的函数，$a$不是$\mathbf{X}$的函数，上面的公式可以根据链式法则迭代使用。
上面的绝大部分公式可以使用$\mathbf{F}(\mathbf{X} + d\mathbf{X}) - \mathbf{F}(\mathbf{X})$计算，取线性部分可以得到，例如：
$$(\mathbf{X} + d\mathbf{X}) (\mathbf{Y} + d\mathbf{Y}) = \mathbf{X}\mathbf{Y} + (d\mathbf{X})\mathbf{Y} + \mathbf{X}d\mathbf{Y} + (d\mathbf{X})(d\mathbf{Y})$$
然后得到$d(\mathbf{X}\mathbf{Y})= (d\mathbf{X})\mathbf{Y}+\mathbf{X}(d\mathbf{Y})$。
计算$d\mathbf{X}^{-1} $，有
$$0 = d\mathbf{I} = d(\mathbf{X}^{-1} \mathbf{X}) = (d(\mathbf{X}^{-1}) \mathbf{X} + \mathbf{X}^{-1} d(\mathbf{X})$$
移项得$d(\mathbf{X}^{-1} ) = - \mathbf{X}^{-1} (d\mathbf{X}) \mathbf{X}^{-1} $
关于迹的公式，有：
$$tr(\mathbf{X} + d\mathbf{X}) - tr(\mathbf{X}) = tr(d\mathbf{X})$$

下面给出导数和微分之间转换的标准公式，我们的目标就是使用上面的公式将一些复杂的公式转换成下面的标准公式。
#### 微分和导数的转换
标准微分公式|等价的导数形式
:-:|:-:
$dy = a\ dx$ | $\frac{dy}{dx} = a$
$dy = \mathbf{a}d\mathbf{x}$ | $\frac{dy}{d \mathbf{x}} = \mathbf{a}$
$dy = tr(\mathbf{A}d\mathbf{A})$ | $\frac{dy}{d \mathbf{X}} = \mathbf{A}$
$d\mathbf{y} = \mathbf{a}dx$ | $\frac{d\mathbf{y}}{d x} = \mathbf{a}$
$d\mathbf{y} = \mathbf{A}d\mathbf{x}$ | $\frac{d\mathbf{y}}{d \mathbf{x}} = \mathbf{A}$
$d\mathbf{Y} = \mathbf{A}dx$ | $\frac{d\mathbf{Y}}{dx} = \mathbf{A}$

有一个很重要的公式是：
$$tr(\mathbf{A}\mathbf{B}) = tr(\mathbf{B}\mathbf{A})$$

#### 示例
$$\frac{d}{d\mathbf{X}} tr(\mathbf{A}\mathbf{X}\mathbf{B}) = tr(\mathbf{A}\mathbf{B})$$
因为：
$$d tr(\mathbf{A}\mathbf{X}\mathbf{B}) = tr(d(\mathbf{A}\mathbf{X}\mathbf{B})) = tr(\mathbf{A}d(\mathbf{X})\mathbf{B}) =  tr(\mathbf{B}\mathbf{A}d(\mathbf{X})) $$
对应$\frac{d\mathbf{Y}}{dx} = \mathbf{A}$。

##### 二次型
计算二次型$\mathbf{x}^T \mathbf{A}\mathbf{x}$的导数，因为$\mathbf{x}^T \mathbf{A}\mathbf{x}$是一个标量，所以可以套上一个$tr$操作：
$$\frac{d(\mathbf{x}^T \mathbf{A}\mathbf{x})}{d\mathbf{x}}= \mathbf{x}^T (\mathbf{A}^T + \mathbf{A})$$
推导：
\begin{align\*}
d(\mathbf{x}^T \mathbf{A}\mathbf{x}) &= tr(d(\mathbf{x}^T \mathbf{A}\mathbf{x})) \\\\
&= tr(d(\mathbf{x}^T) \mathbf{A}\mathbf{x} + \mathbf{x}^T d(\mathbf{A}) \mathbf{x} + \mathbf{x}^T \mathbf{A}d(\mathbf{x})) \\\\
&=  tr(\mathbf{x}^T \mathbf{A}^T  d(\mathbf{x}) + \mathbf{x}^T \mathbf{A}d(\mathbf{x})) \\\\
&= tr(\mathbf{x}^T (\mathbf{A}^T + \mathbf{A})d(\mathbf{x}))\\\\
\end{align\*}
满足$dy = \mathbf{a}d\mathbf{x}$。所以$\mathbf{x}^T \mathbf{A}\mathbf{x}$的导数是$\mathbf{x}^T (\mathbf{A}^T +\mathbf{A})$。



## 参考文献
1.https://en.wikipedia.org/wiki/Matrix_calculus
2.https://zhuanlan.zhihu.com/p/24709748
3.https://zhuanlan.zhihu.com/p/24863977
4.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf
5.http://www.iro.umontreal.ca/~pift6266/A06/refs/minka-matrix.pdf
