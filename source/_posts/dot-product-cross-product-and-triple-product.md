---
title: dot product cross product and triple product
date: 2019-08-28 17:26:35
tags:
 - 点积
 - 叉积
 - 混合积
 - 线性代数
categories: 线性代数
mathjax: true
---

先给出已知条件，两个列向量$a= (x_1,y_1, z_1), w=(x_2,y_2, z_2)$。

## 点积(dot product)
### 定义
点积，又叫数量积，定义为$a\cdot b = x_1 x_2 + y_1y_2 + z_1z_2$。
另一种定义方式是$a\cdot b = \vert a \vert \vert b \vert cos \<a, b\>$`
这两种定义方式实际上是一样的：
![dot_product](dot_product.jpg)

## 叉积(cross product)
### 定义
叉积，又叫向量积，定义为$\vert a\times b \vert = \vert a \vert \vert b \vert sin \<a, b\>$，最后得到一个方向和$a,b$都正交的向量，并且方向符合右手规则。向量积的大小等于$a,b$构成的平行四边形的面积。
### 属性
1. $a\times b, b\times a$是方向相反的
2. $a,b$的叉积垂直于$a$和$b$
3. 任意向量和它自己的叉积为$0$。

## 混合积(trple product)
### 定义
混合积定义为$(a\times b) \cdot c$，它的绝对值等于以$a,b,c$三个向量构成的形状的体积。
### 属性
向量$a\times b$是一个向量，$a\times b$的大小相当于底面积，方向垂直于$a,b$所有的平面，再和$c$点乘，乘上了$c$投影到$a,b$点乘结果所在的直线上，相当于乘上了高。

## 参考文献
1.MIT线性代数公开课
