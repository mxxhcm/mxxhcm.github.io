---
title: determinants（行列式）
date: 2019-08-28 14:45:18
tags:
 - 行列式
 - determinants
 - 线性代数
categories: 线性代数
mathjax: true
---

## 行列式（Determinants）
这一章介绍行列式相关知识，行列式的一些属性等等。
- 矩阵不可逆，行列式为$0$。
- 主元的乘积是行列式。
- 交换任意两行和两列，行列式的符号改变。
- 行列式的绝对值等于这个矩阵描述的space的体积。
- 行列式的计算公式有三个：
    1. 主元公式，就是所有主元的乘积
    2. big formula
    3. cofact formula

## 定义以及性质
行列式用det表示，给出以下的几个属性：
1. $n\times n$的单位矩阵$I$的行列式$det I = I$
$$\begin{vmatrix}1&0\\\\0&1 \end{vmatrix} = 1$$
2. 交换任意两行，行列式符号取反。
$$\begin{vmatrix}a&b\\\\c&d \end{vmatrix} = - \begin{vmatrix}c&d\\\\a&b \end{vmatrix}$$
3. 行列式是每一行的线性函数
$$\begin{vmatrix}ta&tb\\\\c&d \end{vmatrix} = t \begin{vmatrix}a&b\\\\c&d \end{vmatrix}$$
$$\begin{vmatrix}a+a'&b+b'\\\\c&d \end{vmatrix} = \begin{vmatrix}a'&b'\\\\c&d \end{vmatrix}+\begin{vmatrix}a&b\\\\c&d \end{vmatrix}$$

以上的三个属性是行列式的性质，事实上，它们定义了行列式是什么，从这几个基本属性出发，我们能推导出更多的属性。
4. 如果$A$的两行相等，那么$det A=0$，交换两行，还是矩阵$A$，行列式变号，所以行列式只能为$0$。
假设$A = \begin{bmatrix}a&b\\\\a&b \end{bmatrix}$，$\begin{vmatrix}a&b\\\\a&b \end{vmatrix}= - \begin{vmatrix}a&b\\\\a&b \end{vmatrix}$
5. 从某一行减去其他行的倍数，行列式不变
$$\begin{vmatrix}a&b\\\\c- la&d-lb \end{vmatrix}= \begin{vmatrix}a&b\\\\c&d \end{vmatrix} -l \begin{vmatrix}a&b\\\\a&b\end{vmatrix}   = \begin{vmatrix}a&b\\\\c&d \end{vmatrix}$$
6. 某一行为$0$矩阵，行列式为$0$。
$$\begin{vmatrix}0&0\\\\c&d \end{vmatrix} = \begin{vmatrix}c&d\\\\c&d \end{vmatrix} = 0$$
7. 如果$A$是三角矩阵，行列式的值等于对角元素乘积。
$$\begin{vmatrix}a&b\\\\0&d \end{vmatrix} = \begin{vmatrix}a&0\\\\c&d \end{vmatrix} = \begin{vmatrix}a&0\\\\0&d \end{vmatrix} = ad \begin{vmatrix}1&0\\\\0&1 \end{vmatrix} = ad$$
8. 当且仅当$A$不可逆的时候，$det A\neq 0$
$det A = det U$，如果$A$不可逆，$U$中有零行，从$6$我们知道，行列式为$0$。如果$A$可逆，行列式的值等于主元的乘积。
9. 矩阵$AB$的行列式等于矩阵$A$的行列式以及矩阵$B$的行列式。
$$\begin{vmatrix}a&b\\\\c&d \end{vmatrix}\begin{vmatrix}p&q\\\\r&s \end{vmatrix} = \begin{vmatrix}ap+br&aq+bs\\\\cp+dr&cq+ds \end{vmatrix}$$
证明：
对于$2\times 2$的情况，有$\begin{vmatrix}A\end{vmatrix}\begin{vmatrix}B\end{vmatrix} = (ad - bc) ( ps - qr) = (ap + br) (cq+ds) - (aq+bs)(cp+dr) \begin{vmatrix}AB \end{vmatrix}$
当$B$是$A^{-1} $的时候，有$det (A A^{-1}) = det (I) = 1 = det (A) det(A^{-1} )$，所以$det A^{-1} = \frac{1}{det A}$
10. $A^T$和$A$的行列式相同。
$PA=LU, A^T P^T = U^T L^T$，$det L = det L^T = 1, det U = det U^T $，$L$是对角线元素为$1$的对角矩阵，$U$是对角矩阵，$P$是置换矩阵，$P^T P = I$，$det P det P^T = 1$，则$det P = det P^T = 1$，这个为什么？我有点不明明白。最后有$det A = det A^T $。

  
## 行列式的计算
### 主元公式
行列式等于主元的乘积。

### 大公式
$n=2$的情况下：
$$A= \begin{bmatrix} a & b\\\\c&d\\\\\end{bmatrix}$$
$$det A = \begin{vmatrix}a&0\\\\c&d\end{vmatrix}+\begin{vmatrix}0&b\\\\c&d\end{vmatrix} = \begin{vmatrix}a&0\\\\c&0\end{vmatrix}+\begin{vmatrix}a&0\\\\0&d\end{vmatrix}+\begin{vmatrix}0&b\\\\c&0\end{vmatrix} \begin{vmatrix}0&b\\\\0&d\end{vmatrix} = ad - bc$$
$n=3$的情况下，最后有六项不为$0$的取值，$3!= 3\times 2\times 1= 6$
在$n$的情况下，有$n!$个项，将它们加起来求和。

### 主子式（Cofactors）
#### 定义
用$C$表示主子式，用$M_{ij}$表示划去$i$行，$j$列的子矩阵，
$$C_{ij} = (-1)^{i+j} det  M_{ij} $$

#### 计算行列式
行列式可以沿着任意一行或者任意一列，利用主子式进行计算，
沿着第$i$行计算的公式如下：
$$ det A = \sum_{j=1}^n a_{ij} C_{ij}$$
沿着第$j$列计算的公式如下：
$$ det A = \sum_{i=1}^m a_{ij} C_{ij}$$
可以递归下去进行计算。

## 参考文献
1.MIT线性代数公开课视频
