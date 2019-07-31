---
title: 主成分分析(Principal Component Analysis)
date: 2019-01-02 20:51:19
tags:
 - 机器学习
 - 主成分分析
 - PCA
 - 降维
 - 监督学习
categories: 机器学习
mathjax: true
---

## 降维

### 降维的目标
降维可以看做将$p$维的数据映射到$m$维，其中$p\gt m$。

### 降维的目的
1. 维度灾难(curse of dimensionity)
2. 随着维度增加，精确度和效率的退化。
3. 可视化数据
4. 数据压缩
5. 去噪声
...

### 降维的方法
#### 无监督的降维
1. 线性的:PCA
2. 非线性的: GPCA, Kernel PCA, ISOMAP, LLE

#### 有监督的降维
1. 线性的: LDA
2. 非线性的: Kernel LDA


## 主成分分析(PCA)
主成分分析(principal component analysis,PCA)是一个降维工具。PCA使用正交变换(orthogonal transformation)将可能相关的变量的一系列观测值(observation)转换成一系列不相关的变量，这些转换后不相关的变量叫做主成分(principal component)。第一个主成分有着最大的方差，后来的主成分必须和前面的主成分正交，然后最大化方差。或者PCA也可以看成根据数据拟合一个$m$维的椭球体(ellipsoid)，椭球体的每一个轴代表着一个主成分。
上课的时候，老师给出了五种角度来看待PCA，分别是信息保存，投影，拟合，嵌入(embedding)，mainfold learning。本文首先从保存信息的角度来给出PCA的推理过程，其他的几种方法就随缘了吧。。。

### 信息保存(preserve information)
#### 目标
从信息保存的角度来看PCA的目标是用尽可能小的空间去存储尽可能多的信息。一般情况下，信息用信息熵$-\int p lnp$来表示，如果这里使用信息熵的话，不知道信息的概率表示，一般不知道概率分布的情况下就采用高斯分布，带入高斯分布之后得到$\frac{1}{2}log(2\pi e\sigma^2)$，其中$2\pi e$都是常量，只剩下方差。给出一堆数据，直接计算信息熵是行不通的，但是计算方差是可行的，而方差和信息熵是有联系的，所以可以考虑用方差来表示信息。考虑一下降维前的$p$维数据$x$和降维后的$m$维数据$z$方差之间的关系，$var(z)?var(x)$，这里$z$和$x$的方差维度是不同的，所以不能相等，这里我们的目标就是最大化$z$的方差。方差能解释变化，方差越大，数据的变化就越大，越能包含信息。PCA的目标就是让降维后的数据方差最大。

#### 线性PCA过程

##### 目标函数
给定$n$个观测数据$x_1,x_2,\cdots,x_n \in R^p$，形成一个观测矩阵$X,X\in R^{p\times n}$，我们的目标是将这样一组$p$维的数据转换成$m$维的数据。线性PCA是通过线性变换(matrix)来实现的，也就是我们要求一个$p\times m$的矩阵$V$，将原始的$X$矩阵转换成$Z$矩阵，使得
$$Z_{m\times n}= V_{p\times m}^{T}X_{p\times n},$$
其中$V\in R^{p\times m}$, $v_i=\begin{bmatrix}v_{1i}\\v_{2i}\\ \vdots\\v_{pi}\end{bmatrix}$, $V = \begin{bmatrix}v_{11}&v_{12}&\cdots&v_{1m}\\v_{21}&v_{22}&\cdots&v_{2m}\\ \vdots&\vdots&\cdots&\vdots\\v_{p1}&v_{p2}&\cdots&v_{pm}\end{bmatrix}=\begin{bmatrix}v_1&v_2&\cdots&v_m\end{bmatrix}$, $V^T = \begin{bmatrix}v_{11}&v_{21}&\cdots&v_{p1}\\v_{12}&v_{22}&\cdots&v_{p2}\\ \vdots&\vdots&\cdots&\vdots\\v_{1m}&v_{2m}&\cdots&v_{pm}\end{bmatrix}=\begin{bmatrix}v_1^T\\v_2^T\\ \vdots\\v_m^T\end{bmatrix}$。
所以就有：
\begin{align\*}
z_1 &= v_1^Tx_j\\
&\cdots\\
z_k &= v_k^Tx_j\\
&\cdots\\
z_m &= v_m^Tx_j
\end{align\*}
其中，$z_1,\cdots,z_m$是标量，$v_1^T,\cdots, v_m^T$是$1\times p$的向量，$x_j$是一个$p\times 1$维的观测向量，而我们有$n$个观测向量，所以随机变量$z_k$共有$n$个可能取值：
$$z_{k} = v_k^Tx_i= \sum_{i=1}^{p}v_{ik}x_{ij}, j = 1,2,\cdots,n$$
其中$x_i$是观测矩阵$X$的第$i$列，$X\in R^{p\times n}$。

##### 协方差矩阵
离散型随机变量$X$($X$的取值等可能性)方差的计算公式是：
$$var(X) = E[(X-\mu)^2] = \frac{1}{n}\sum_{i=1}^n(x_i-\mu)^2,$$
其中$\mu$是X的平均数，即$\mu = \frac{1}{n}\sum_{i=1}^nx_i$。

让$z_k$的方差最大即最大化：
\begin{align\*}
var(z_1) &=E(z_1,\bar{z_1})^2\\
&=\frac{1}{n}\sum_{i=1}^n(v_1^Tx_i - v_1^T\bar{x_i})^2\\
&=\frac{1}{n}\sum_{i=1}^n(v_1^Tx_i - v_1^T\bar{x_i})(v_1^Tx_i - v_1^T\bar{x_i})^T\\
&=\frac{1}{n}\sum_{i=1}^nv_1^T(x_i - \bar{x_i})(x_i - \bar{x_i})^Tv_1\\
\end{align\*}
其中$x_i=\begin{bmatrix}x_{1i}\\x_{2i}\\ \vdots\\x_{pi}\end{bmatrix}$,$\bar{x_i}=\begin{bmatrix}\bar{x_{1i}}\\\bar{x_{2i}}\\ \vdots\\\bar{x_{pi}}\end{bmatrix}$,$x_i$是$p$维的，$x_i^p$也是$p$维的，$(x_i-\bar{x_i})$是$p\times 1$维的，$(x_i -\bar{x_i})^T$是$1\times p$维的。
令$S=\frac{1}{n}\sum_{i=1}^n(x_i -\bar{x_i})(x_i-\bar{x_i})^T$，$S$是一个$p\times p$的对称矩阵，其实$S$是一个协方差矩阵。这个协方差矩阵可以使用矩阵$X$直接求出来，也可以通过对$X$进行奇异值分解求出来。
如果使用奇异值分解的话，首先对矩阵$X$进行去中心化，即$\bar{x_i}=0$，则：
\begin{align\*}
S &= \frac{1}{n}\sum_{i=1}^Tx_ix_i^T\\
&=\frac{1}{n}X_{p\times n}X_{n\times p}^T
\end{align\*}
$X=U\Sigma V^T$
$XX^T=U\Sigma V^TV\Sigma U^T = U\Sigma_1^2U^T$
$X^TX =V\Sigma U^TU\Sigma V^T = V\Sigma_2^2V^T$ 
$S=\frac{1}{n}XX^T=\frac{1}{n}U\Sigma^2U^T$

##### 拉格朗日乘子法
将$S$代入得：
$$var(z_1) = v_1^TSv_1,$$
接下来的目标是最大化$var(z_1)$，这里要给出一个限制条件，就是$v_1^Tv_1 = 1$，否则的话$v_1$无限大，$var(z_1)$就没有最大值了。
使用拉格朗日乘子法，得到目标函数：
$$L=v_1^TSv_1 - \lambda (v_1^Tv_1 -1)$$
求偏导，令偏导数等于零得：
\begin{align\*}
\frac{\partial{L}}{\partial{v_1}}&=2Sv_1 - 2\lambda v_1\\
&=2(S-\lambda) v_1\\
&=0
\end{align\*}
即$Sv_1 = \lambda v_1$，所以$v_1$是矩阵$S$的一个特征向量(eigenvector)。所以：
$$var(z_1) = v_1^TSv_1 = v_1^T\lambda v_1 = \lambda v_1^Tv_1 = \lambda,$$
第一个主成分$v_1$对应矩阵$S$的最大特征值。
##### 其他主成分
对于$z_2$,同理可得：
$var(z_2) = v_2^TSv_2$，
但是这里要加一些限制条件$v_2^Tv_2=1$，除此以外，第2个主成分还有和之前的主成分不相关，即$cov[z_1,z_2]=0$,或者说是$v_1^Tv_2=0$，证明如下。
\begin{align\*}
cov[z_1,z_2] &=E[(z_1-\bar{z_1})(z_2-\bar{z_2})]\\
&=\frac{1}{n}(v_1^Tx_i - v_1^T\bar{x_i})(v_2^Tx_i-v_2^T\bar{x_i})\\
&=\frac{1}{n}v_1^T(x_i-\bar{x_i})(x_i-\bar{x_i})v_2\\
&=\frac{1}{n}v_1^TSV_2\\
&=\frac{1}{n}\lambda v_1^Tv_2\\
&=0
\end{align\*}
维基百科上是通过将数据减去第一个主成分之后再最大化方差，这两种理解方法都行。
所以拉格朗日目标函数就成了：
$$L=v_1^TSv_1 - \lambda (v_1^Tv_1 -1) -\beta v_2^Tv_1$$
求导，令导数等于零得：
$$\frac{\partial{L}}{\partial{v_1}}=2Sv_2 - 2\lambda v_2 - \beta v_1 = 0$$
而$v_1$和$v_2$不相关，所以$\beta=0$，所以$Sv_2 = \lambda v_2$，即$v_2$也是矩阵$S$的特征向量，但是最大的特征值对应的特征向量已经被$v_1$用了，所以$v_2$是第二大的特征值对应的特征向量。
同理可得第$k$个主成分是$S$的第$k$大特征值对应的特征向量。

但是这种理解方法没有办法推广到非线性PCA。接下来的集中理解方式可以由线性PCA开始，并且可以推广到非线性PCA。

### 函数拟合
#### 线性PCA过程

#### 非线性PCA过程
##### 广义主成分分析(Generalized PCA,GPCA)
刚才讲的PCA是线性PCA，是拟合一个超平面(hyperplane)的过程，但是如果数据不是线性的，比如说是一个曲面$x^2+y^2+z=0$，这样子线性PCA就不适用了，可以稍加变化让其依然是可以用的。比如$x+y+1=0$可以看成$\begin{bmatrix}a&b&c\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix}$，而$x^2+y^2+z=0$可以看成$\begin{bmatrix}a&b&c\end{bmatrix}\begin{bmatrix}x^2\\y^2\\z\end{bmatrix}$。

如果原始数据是非线性的，我们可以通过多个特征映射函数$\Phi$从原始数据提取非线性特征（也可看成升维，变成高维空间中数据，在高维中可以看成是线性的），然后利用线性PCA对非线性特征进行降维。例如：
假设$x=[x_1,x_2,x_3]^T \in R^3$，按照转换函数$v(x) = [x_1^2,x_1x_2,x_1x_3,x_2^2,x_2x_3,x_3^2$将其转换成$R^6$中的特征，接下来使用线性PCA对这些非线性特征进行降维。

给定一个函数$\Phi$将$p$维数据映射到特征空间$F$中，即$\Phi:R^p\rightarrow F,\mathbf{x}\rightarrow X$。我们可以通过计算协方差矩阵$C_F = \frac{\Phi\Phi^T}{n}$,即$C_F = \frac{1}{n}\sum_{i=1}^{n}\phi(x_i)\phi(x_i)^T$，然后对协方差矩阵$C_F$进行特征值分解$C_Fx=\lambda x$就可以求解，这里我们假设空间$F$中的数据均值为$0$，即$E[\Phi(x)] = 0$。

### 嵌入(embedding)，保距离
#### 核函数技巧(Kernel trick)
在GPCA中，如果不知道$\Phi$的话，或者$\Phi$将数据映射到了无限维空间中，就没有办法求解了。这里就给出了一个假设，假设低维空间中$x_i,x_j$的点积(dot product)可以通过一个函数计算，将$x_i,x_j$的点积记为$K_{ij}$，则：
$$K_{ij} = \lt \phi(x_i),\phi(x_j) \gt = k(x_i,x_j)$$
其中$k()$是一个函数，比如可以取高斯函数，$k(x,y) = e^{\frac{(\Vert x-y\Vert)^2}{2\sigma^2}}$，我们叫它核函数(kernel function)。
这样即使我们不知道$\Phi$，也可以计算点积，直接使用核函数计算。

#### dot PCA
给定原始数据$X_D = [x_1,\cdots,x_n],x_i\in R^p$，假定$\hat{x}=0$，那么$X_D$的协方差矩阵：
\begin{align\*}
S&= \frac{\sum_{i=1}^n(x_i-\bar{x})(x_i-\bar{x})^T}{n}\\ 
&= \frac{\sum_{i=1}^n(x_i-0)(x_i-0)^T}{n}\\ 
&= \frac{\sum_{i=1}^n(x_i)(x_i)^T}{n}\\
&= \frac{\begin{bmatrix}x_1&\cdots&x_n\end{bmatrix}\begin{bmatrix}x_1\\\vdots\\x_n\end{bmatrix}}{n}\\
&= \frac{X_DX_D^T}{n}\\
&= Cov(X_D, X_D)
\end{align\*}
即$S=\frac{X_DX_D^T}{n}$，而对$X_D$做奇异值分解，有$X_D = V\Sigma U^T$，所以$S = \frac{V\Sigma^2 V^T}{n}$，其中$U$是$S$的特征值矩阵，则：$Z' = V^T X'$，其中$V\in R^{p\times m}$，$X'$是新的样本数据。

这里我们推导一下点积和PCA的关系，即假设我们有$K = Dot(X_D,X_D) = X_D^TX_D$，则$K=U^T\sigma^2U$，而我们根据奇异值分解$X_D = V\Sigma U^T$可以得到$U$和$V$的关系，即$V=X_DU\Sigma^{-1}$，对$K$进行特征值分解，可以求得$U$和$\Sigma$，所以来了一个新的样本$X'$，
$$Z' = V^TX' = D^{-1}U^TX_D^TX' = D^{-1}U^T\lt X_D,X' \gt.$$
事实上，这里$X'$是已知的，可以直接计算协方差，但是这里是为了给Kernel PCA做引子，所以，推导的过程中是没有用到$X$的，只用到了$X$的点积，在测试的时候会用到$X'$。

##### Kernel PCA
Kernel PCA就是将Kernel trick应用到了dot PCA中，由Kernel trick得$K = \Phi^T\Phi$，$K=U\Sigma^2U^T$，则
$$V = \Phi U\Sigma^{-1} = \Phi U diag(1/sqrt(\lambda_1),1/sqrt(\lambda_2),\cdots)$$
但是我们求不出来$V$，因为$\Phi$不知道，但是可以让$V$中的$\Phi$和样本$X'$中的$\Phi$在一起，就可以计算了，即
$$Z' = V^T\phi(X') = \Sigma^{-1}U\Phi\phi(X') = \Sigma^{-1}UK(X,X')$$

### 流形学习(manifold)
#### 线性
##### PCA
##### MDS
#### 非线性
##### LLE
##### ISOMAP

## 线性判断分析(Fisher linear discrimiant analysis,LDA)
### 线性LDA
#### 两类
##### 示例
#### C类(C$\gt 2$)
两维的问题是通过将原始数据投影到一维空间进行分类，而$C$维的问题则是将原始数据投影到$C-1$空间进行分类，通过一个投影矩阵$W=\begin{bmatrix}w_1&\cdots&w_{C-1}\end{bmatrix}$将$C$维的$x$投影到$C-1$维，得到$y=\begin{bmatrix}y_1&\cdots&y_{C-1}\end{bmatrix}$，即$y_i = w_i^Tx\Rightarrow y = W^Tx$。
##### 示例
### 不足
- 最多投影到$C-1$维特征空间。
- LDA是参数化的方法，它假设数据服从单高斯分布，并且所有类的协方差都是等价的。对于多个高斯分布，线性的LDA是无法分开的。
- 当数据之间的差异主要通过方差而不是均值体现的话，LDA就会失败(fail)。如下图
![figure]()

### Kernel LDA

## PCA和LDA区别和联系
PCA是一个无监督的降维方法，通过最大化降维后数据的方差实现；LDA是一个有监督的降维方法，通过最大化类可分性实现(class discrimnatory)。

## 参考文献
1.https://en.wikipedia.org/wiki/Principal_component_analysis
2.https://en.wikipedia.org/wiki/Variance
3.https://sebastianraschka.com/faq/docs/lda-vs-pca.html
