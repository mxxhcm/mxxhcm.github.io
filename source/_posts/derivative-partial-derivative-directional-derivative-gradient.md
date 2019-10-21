---
title: gradient, directional, derivative derivative, partial derivative
date: 2019-09-13 09:56:12
tags:
 - 梯度
 - 导数
 - 偏导数
 - 方向导数
 - 高等数学
categories: 高等数学
---

## 导数
导数表示函数在该点的变化率。
$$f'(x) = lim_{\Delta x\rightarrow 0}\frac{\Delta y}{\Delta x} = lim_{\Delta x\rightarrow 0} \frac{f(x+\Delta x) - f(x)}{ \Delta x}$$
更直接的来说，导数表示自变量无穷小时，函数值的变化与自变量变化的比值，几何意义是该点的切线。物理意义表示该时刻的瞬时变化率。

## 偏导数
偏导数是多元函数沿着坐标轴的变化率。
在一元函数中，只有一个自变量，也就是只存在一个方向上的变化率，叫做导数。对于多元函数，有两个及以上的自变量。从导数到偏导数，相当于从曲线到了曲面，曲线上的一点只有一条切线，而曲面上的一点有无数条切线。我们把沿着坐标轴上的切线的斜率叫做偏导数。

## 方向导数
多元函数沿着任意向量的变化率。

## 梯度
多元函数取最大变化率时的方向向量。
对于三元函数，计算公式：
$$\nabla f  = \frac{\partial f}{\partial x}i+\frac{\partial f}{\partial y}j + \frac{\partial f}{\partial z}j$$
梯度到底是行向量还是列向量？取决于使用什么layout。
梯度的公式是$\frac{\partial{y}}{\partial\mathbf{x}}$，
如果使用numerator layout，梯度是行向量；
如果使用denominator layout，梯度是列向量。

## 全微分
函数从$A$点到离它无穷近的$B$点的变化量。
对于二元函数，定义为
$$dz = \frac{\partial f}{\partial x}dx+\frac{\partial f}{\partial y}dy$$

## 区别和联系
导数，偏导数和方向导数都是个向量，它们其实都是一个概念。偏导数和导数都是方向导数的特殊情况。
梯度是个向量，它指向最陡峭的上升方向，这个时候方向导数最大。

## 参考文献
1.https://www.zhihu.com/question/36301367/answer/142096153
2.https://math.stackexchange.com/questions/661195/what-is-the-difference-between-the-gradient-and-the-directional-derivative
