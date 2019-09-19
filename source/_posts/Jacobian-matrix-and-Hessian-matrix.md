---
title: 雅克比矩阵和海塞矩阵
date: 2019-09-10 19:22:54
tags:
 - 线性代数
 - 雅克比矩阵
 - 海塞矩阵
categories: 线性代数
---

## 雅克比矩阵和海塞矩阵对比
1. 雅克比矩阵是一阶导数，海塞矩阵是二阶导数。
2. 雅克比矩阵要求函数的输入是向量，输出也是向量，而海塞矩阵要求输入是向量，输出是标量。
3. 雅克比矩阵不一定是方阵（当输入和输出的维度相等时是方阵），但是海塞矩阵一定是方阵。当函数的输出是标量的时候，雅克比矩阵退化成了梯度向量。
4. 海塞矩阵可以看成梯度的雅克比矩阵。

## 雅克比矩阵
定义：$f:\mathbb{R}^n \rightarrow \mathbf{R}^m $，即一个函数的输入和输出都是向量，计算输出向量对于输入向量的偏导数（详情可见[矩阵求导](https://mxxhcm.github.io/2019/09/12/matrix-calculus/))，得到一个$m\times n$的矩阵（numerator layout），这就是雅克比矩阵。
$$J = \begin{bmatrix}\frac{\partial \mathbf{y}}{\partial x_1} & \cdots & \frac{\partial \mathbf{y}}{\partial x_n}\end{bmatrix} = \begin{bmatrix} \frac{\partial \mathbf{y_1}}{\partial x_1} & \cdots & \frac{\partial \mathbf{y_1}}{\partial x_n}\\\\ \cdots \\\\ \frac{\partial \mathbf{y_m}}{\partial x_1}\cdots & \cdots & \frac{\partial \mathbf{y_m}}{\partial x_n}\end{bmatirx}$$



## 海塞矩阵
定义：$f:\mathbb{R}^n \rightarrow \mathbf{R} $，即一个函数的输入是向量，输出是标量时，计算输出对于输入向量的二阶导数（详情可见[矩阵求导](https://mxxhcm.github.io/2019/09/12/matrix-calculus/))，得到一个$n\times n$的矩阵（numerator layout），这就是雅克比矩阵。
$$ H = \begin{bmatrix} \frac{\partial^2 \mathbf{y}}{\partial x_1^2 }\cdots & \cdots & \frac{\partial^2 \mathbf{y}}{\partial x_1 x_n}\\\\ \cdots \\\\ \frac{\partial^2 \mathbf{y}}{\partial x_nx_1}\cdots & \cdots & \frac{\partial^2 \mathbf{y}}{\partial x_nx_n}\end{bmatrix}$$


## 参考文献
1.https://zhuanlan.zhihu.com/p/67521774
