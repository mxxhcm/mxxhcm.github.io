---
title: linear algebra Gram-Schmidt 正交化
date: 2019-11-01 09:39:44
tags:
 - 线性代数
categories: 线性代数
---

## Gram-Schmidi正交化
这一章属于[正交]()的内容，但是因为很重要，就单独拎出来再说一遍。
Gram-Schmidt正交化过程就相当于是在不断的进行投影，这个方法的想法是从$n$个独立的column vector出发，构建$n$个正交向量，然后再单位化。拿$3$个过程举个例子。用$a,b,c$表示初始的$3$个独立向量，$A,B,C$表示三个正交向量，$q_1, q_2,q_3$表示三个正交单位向量。
第一个正交向量，直接对第一个向量单位化
$$A=a, q_1 = \frac{A}{\vert A\vert}$$
第二个正交向量，将第二个向量投影到第一个向量上，计算出一个和第二个向量正交的向量。
$$B=b-\frac{A^T B}{A^T A}A , q_2 = \frac{B}{\vert B\vert}$$
第三个正交向量，将第三个向量分别投影到第一个和第二个正交向量上，计算处第三个正交向量。
$$C=c - \frac{A^T C}{A^T A}A - \frac{B^T C}{B^T B}B , q_2 = \frac{C}{\vert C\vert}$$
![gram_schmidi](gram_schmidi.jpg)


## 参考文献
1.MIT线性代数公开课
