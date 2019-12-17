---
title: singular value decomposition（奇异值分解）
date: 2019-01-03 15:19:54
tags:
 - 线性代数
categories: 线性代数
mathjax: true
---

## 特征值分解(eigen value decomposition)
要谈奇异值分解，首先要从特征值分解(eigen value decomposition, EVD)谈起。
矩阵的作用有三个：一个是旋转，一个是拉伸，一个是平移，都是线性操作。如果一个$n\times n$方阵$A$对某个向量$x$只产生拉伸变换，而不产生旋转和平移变换，那么这个向量就称为方阵$A$的特征向量(eigenvector)，对应的伸缩比例叫做特征值(eigenvalue)，即满足等式$Ax = \lambda x$。其中$A$是方阵，$x$是方阵$A$的一个特征向量，$\lambda$是方阵$A$对应特征向量$x$的特征值。
假设$S$是由方阵$A$的$n$个线性无关的特征向量构成的方阵，$\Lambda$是方阵$A$的$n$个特征值构成的对角矩阵，则$A=S\Lambda S^{-1}$，这个过程叫做对角化过程。
证明：
因为$Ax_1 = \lambda_1 x_1,\cdots,Ax_n = \lambda_n x_n$,
所以
\begin{align\*}AS &= A\begin{bmatrix}x_1& \cdots&x_n\end{bmatrix}\\\\
&=\begin{bmatrix} \lambda_1x_1&\cdots&\lambda x_n\end{bmatrix}\\\\
&= \begin{bmatrix}x_1& \cdots&x_n\end{bmatrix} \begin{bmatrix}\lambda_1& & &\\\\&\lambda_2&&\\\\&&\cdots&\\\\&&&\lambda_n\end{bmatrix}\\
&= S\Lambda
\end{align\*}
所以$AS=S\Lambda, A=S\Lambda S^{-1}, S^{-1}AS=\Lambda$。
若方阵$A$为对称矩阵，矩阵$A$的特征向量是正交的，将其单位化为$Q$，则$A=Q\Lambda Q^T$，这个过程就叫做特征值分解。

## 奇异值分解(singular value decomposition)
特征值分解是一个非常好的分解，因为它能把一个方阵分解称两类非常好的矩阵，一个是正交阵，一个是对角阵，这些矩阵都便于进行各种计算，但是它对于原始矩阵的要求太严格了，必须要求矩阵是对称正定矩阵，这是一个很苛刻的条件。所以就产生了奇异值分解，奇异值分解可以看作特征值分解在$m\times n$维矩阵上的推广。对于对称正定矩阵来说，有特征值，对于其他一般矩阵，有奇异值。

奇异值分解可以看作将一组正交基映射到另一组正交基的变换。普通矩阵$A$不是对称正定矩阵，但是$AA^T $和$A^TA $一定是对称矩阵，且至少是半正定的。从对$A^TA $进行特征值分解开始，$A^T A=V\Sigma_1V^T $，$V$是一组正交的单位化特征向量$\{v_1,\cdots,v_n\}$，则$Av_1,\cdots,Av_n$也是正交的。
证明：
\begin{align\*}Av_1\cdot Av_2 &=(Av_1)^T Av_2\\\\
&=v_1^T A^T Av_2\\\\
&=v_1^T \lambda v_2\\\\
&=\lambda v_1^T v_2\\\\
&=0
\end{align\*}
所以$Av_1,Av_2$是正交的，同理可得$Av_1,\cdots,Av_n$都是正交的。
而：
\begin{align\*}
Av_i\cdot Av_i &= v_i^T A^T Av_i\\\\
&=v_i \lambda v_i\\\\
&=\lambda v_i^2\\\\
&=\lambda
\end{align\*}
将$Av_i$单位化为$u_i$，得$u_i = \frac{Av_i}{|Av_i|} = \frac{Av_i}{\sqrt{\lambda_i}}$，所以$Av_i = \sqrt{\lambda_i}u_i$。
将向量组$\{v_1,\cdots,v_r\}$扩充到$R^n $中的标准正交基$\{v_1,\cdots,v_n\}$，将向量组$\{u_1,\cdots,u_r\}$扩充到$R^n $中的标准正交基$\{u_1,\cdots,u_n\}$，则$AV = U\Sigma$，$A=U\sigma V^T $。

事实上，奇异值分解可以看作将行空间的一组正交基加上零空间的一组基映射到列空间的一组正交基加上左零空间的一组基的变换。对一矩阵$A,A\in \mathbb{R}^{m\times n} $，若$r(A)=r$，取行空间的一组特殊正交基$\{v_1,\cdots,v_r\}$，当矩阵$A$作用到这组基上，会得到另一组正交基$\{u_1,\cdots,u_r\}$，即$Av_i = \sigma_iu_i$。
矩阵表示是：
\begin{align\*}
AV &= A\begin{bmatrix}v_1&\cdots&v_r\end{bmatrix}\\\\
&= \begin{bmatrix}\sigma_1u_1 & \cdots & \sigma_ru_r\end{bmatrix}\\\\
&= \begin{bmatrix}u_1&u_2&\cdots&u_r\end{bmatrix}\begin{bmatrix}\sigma_1&&&\\\\&\sigma_2&&\\\\&&\cdots&\\\\&&&\sigma_n\end{bmatrix}\\\\
&=U\Sigma
\end{align\*}
其中$A\in \mathbb{R}^{m\times n}, V\in \mathbb{R}^{n\times r},U\in \mathbb{R}^{m\times r}, \Sigma \in \mathbb{R}^{r\times n}$。
当有零空间的时候，行空间的一组基是$r$维，加上零空间的$n-r$维，构成$R^n $空间中的一组标准正交基。列空间的一组基也是$r$维的，加上左零空间的$m-r$维，构成$R^m $空间的一组标准正交基。零空间中的向量在对角矩阵$\Sigma$中体现为$0$，
则$A=U\Sigma V^{-1} $，$V$是正交的，所以$A=U\Sigma V^T $，其中$V\in \mathbb{R}^{n\times n}, U\in \mathbb{R}^{m\times m}, \Sigma \in \mathbb{R}^{m\times n}$。

$A=U\Sigma V^T $,
$A^T = V\Sigma^T U^T $,
$AA^T = U\Sigma V^T V\Sigma^T U^T $,
$A^T A = V\Sigma^T U^T U\Sigma V^T $
对$A A^T $和$A^T A$作特征值分解，则$A A^T = U\Sigma_1U^T $,$A^T A=V\Sigma_2V^T $，所以对$AA^T $作特征值分解求出来的$U$和对$A^T A$作特征值分解求出来的$V$就是对$A$作奇异值分解求出来的$U$和$V$，$AA^T $和$A^T A$作特征值分解求出来的$\Sigma$的非零值是相等的，都是对$A$作奇异值分解的$\Sigma$的平方。

### $A^T A$和$AA^T $的非零特征值是相等的
证明：对于任意的$m\times n$矩阵$A$，$A^T A$和$AA^T $的非零特征值相同的。 设$A^T A$的特征值为$\lambda_i$，对应的特征向量为$v_i$，即$A^T Av_i = \lambda_i v_i$。
则$AA^T Av_i = A\lambda_iv_i = \lambda_i Av_i$。
所以$AA^T $的特征值为$\lambda_i$，对应的特征向量为$Av_i$。
因此$A^T A$和$AA^T $的非零特征值相等。

### 几何意义
对于任意一个矩阵，找到其行空间(加上零空间)的一组正交向量，使得该矩阵作用在该向量序列上得到的新的向量序列保持两两正交。奇异值的几何意义就是这组变化后的新的向量序列的长度。

### 物理意义
奇异值往往对应着矩阵隐含的重要信息，且重要性和奇异值大小正相关。每个矩阵都可以表示为一系列秩为$1$的“小矩阵”的和，而奇异值则衡量了这些秩一矩阵对$A$的权重。
奇异值分解的物理意义可以通过图像压缩表现出来。给定一张$m\times n$像素的照片$A$，用奇异值分解将矩阵分解为若干个秩一矩阵之和，即：
\begin{align\*}
A&=\sigma_1 u_1v_1^T +\sigma_2 u_2v_2^T +\cdots+\sigma_r u_rv_r^T\\\\
&= \begin{bmatrix}u_1&u_2&\cdots&u_r\end{bmatrix}\begin{bmatrix}\sigma_1&&&\\&\sigma_2&&\\&&\cdots&\\&&&\sigma_n\end{bmatrix}\begin{bmatrix}v_1^T\\v_2^T\\ \vdots\\v_r^T\end{bmatrix}\\\\
&=U\Sigma V^T
\end{align\*}

这个也叫部分奇异值分解。其中$V\in R^{r\times n}, U\in R^{m\times r}, \Sigma \in R^{r\times r}$。因为不含有零空间和左零空间的基，如果加上零空间的$n-r$维和左零空间的$m-r$维，就是奇异值分解。
较大的奇异值保存了图片的主要信息，特别小的奇异值有时可能是噪声，或者对于图片的整体信息不是特别重要。做图像压缩的时候，可以只取一部分较大的奇异值，比如取前八个奇异值作为压缩后的图片：
$$A = \sigma_1 u_1v_1^T +\sigma_2 u_2v_2^T + \cdots + \sigma_8 u_8v_8^T$$
现实中常用的做法有两个：
1. 保留矩阵中$90%$的信息：将奇异值平方和累加到总值的%90%为止。
2. 当矩阵有上万个奇异值的时候，取前面的$2000$或者$3000$个奇异值。。

## 参考文献(references)
1.Gilbert Strang, MIT Open course：Linear Algebra
2.https://www.cnblogs.com/pinard/p/6251584.html
3.http://www.ams.org/publicoutreach/feature-column/fcarc-svd
4.https://www.zhihu.com/question/22237507/answer/53804902
5.http://charleshm.github.io/2016/03/Singularly-Valuable-Decomposition/
