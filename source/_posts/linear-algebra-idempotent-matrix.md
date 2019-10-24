---
title: linear algebra idempotent matrix
date: 2019-08-24 13:07:15
tags:
 - 线性代数
 - idempotent matrix
categories: 线性代数
mathjax: ture
---

## Idempotent Matirx（幂等矩阵）
**定义**满足$A^2 = A$的矩阵叫做idempotent matirx（幂等矩阵）。
幂等矩阵必须是方阵。

## 示例
$$\begin{bmatrx} 4&-1\\\\12&-3\end{bmatrix}$$

## 属性
- 除了identity matrix，所有的idempotent matrix都是singular。如果它是方阵，那么它的determiant是$0$。
- 如果M是幂等矩阵，那么M-I也一定是幂等矩阵。
$$(I-M)(I-M) =  I^2 - 2IM + M^2 = I - M$$
- 幂等矩阵的特征值只能是$0$或者$1$
证明： 如果$A$是idempotent, $\lambda$是一个特征值，$v$是对应的特征向量。
$$\lambda v = Av = AAv = \lambda Av = \lambda^2 v$$
因为$v\neq 0$，所以$\lambda=0, or 1$。
- 幂等矩阵可对角化
- 幂等矩阵的迹等于幂等矩阵的秩
- 方阵零矩阵和单位矩阵都是幂等矩阵。


## 参考文献
1.https://www.statisticshowto.datasciencecentral.com/idempotent-matrix/
2.http://people.stat.sfu.ca/~lockhart/richard/350/08_2/lectures/Theory/web.pdf
