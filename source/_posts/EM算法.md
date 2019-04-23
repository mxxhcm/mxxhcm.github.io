---
title: EM(Expectation Maximization)算法
date: 2019-01-21 10:22:45
tags:
- 机器学习
- 非监督学习
- 最大似然估计
- EM
- 期望最大化
- Jensen不等式
categories: 机器学习
mathjax: true
---

## 引言(Introduction)
### 什么是期望最大化算法
期望最大化算法(Expectation Maximization,EM)，是利用参数估计的迭代法求解最大似然估计的一种方法。
### EM和MLE关系
MLE的目标是求解已知分布类型的单个分布的参数。
EM的目标是求解已知分布类型的多个混合分布的参数。
一般我们用到的极大似然估计都是求某种已知分布类型的单个分布的参数，如求高斯分布的均值和方差；而EM算法是用来求解已知分布类型，多个该已知类型分布的混合分布的参数，这句话听起来可能有些拗口，举个最常见的例子，高斯混合分布参数的求解，这个混合分布都是高斯分布，只是每个分布的参数不同而已。如果一个高斯分布，一个卡方分布是没有办法求解的。
### 为什么叫它EM算法
因为这个算法总共有两个迭代步骤，E步和M步。第一步是对多个分布求期望，固定每一个分布的参数，计算出混合分布的参数，即E步，第二步是对这个混合分布利用最大似然估计方法进行参数估计，即M步。


## 推理过程

假设我们要求一个混合分布p的参数$\theta$，比如校园内男生和女生的身高参数，显然，男生和女生的身高服从的分布类型是相同的，但是参数是不一样的。这里通过引入一个隐变量$z$，求解出对应不同$z$取值的参数$\theta$的值。
\begin{align\*}
p(x|\theta) &= \sum_zp(x,z|\theta)\\
&=\sum_zp(z|\theta)p(x|\theta, z) \tag{0}
\end{align\*}
如果我们假设男女生的身高分布是一个高斯混合模型，现在要求它的参数$\theta$。混合模型的表达式可以写为： 
\begin{align\*}
p(x|\theta) &= \sum_zw(z)N(x|\mu_z,\sigma_z)\\
&=\sum_zp(z|\theta)p(x|\theta,z)
\end{align\*}
其中$\sum_zw(z) = 1,\theta=\{w, \mu, \sigma\}$，如果用最大似然估计来解该问题的话，log函数内有和式，不好优化，所以就要换种方法。
观测数据：$x=(x_1,\cdots, x_N)$
对应的隐变量：$z=(z_1,\cdots, z_N)$，$z_i$有$c$种取值。

\begin{align\*}
l(\theta;x) &= log p(x|\theta) \tag{1}\\
&= log\prod_{i=1}^N\ p(x_i|\theta) \tag{2}\\
&= \sum_{i=1}^Nlog\ p(x_i|\theta) \tag{3}\\
&= \sum_{i=1}^Nlog\sum_zp(x_i,z|\theta) \tag{4}\\
\end{align\*}
这里式子(4)中$\sum_zp(x,z|\theta)$该怎么变形，因为现在解不出来了。
最开始我想的是使用条件概率进行展开，即：
$$\sum_zp(x_i, z|\theta) = \sum_zp(x_i|z, \theta)p(z|\theta)$$
但是如果展开成这样子，就变成了文章开头给出的式子(0)，并没有什么用，不能继续化简了。
所以就对式子(4)做个变形
\begin{align\*}
&\ \ \ \ \sum_{i=1}^Nlog\sum_zp(x_i,z|\theta) \tag{4}\\
&= \sum_{i=1}^Nlog\sum_zq(z|x_i)\frac{p(x_i,z|\theta)}{q(z|x_i)}, \ \ s.t.\sum_zq(z|x_i)=1 \tag{5}\\
&\ge \sum_{i=1}^N \underbrace{\sum_zq(z|x_i)log\frac{p(x_i,z|\theta)}{q(z|x_i)}}_{L(q,\theta)},\ \ s.t. \sum_zq(z|x_i)=1 \tag{6}\\
\end{align\*}
第(4)步到第(5)步引入了一个分布$q(z|x)$，就是给定一个观测数据$x$，隐变量$z$取值的概率分布。注意，$q(z)$是一个函数，但是给定$x$之后，$q(z|x)$是一个变量。然后因为变形之后还是没有求解，就利用杰森不等式做了缩放，将$log(sum())$变成了$sum(log())$，就变成了(6)式。
这里使用Jensen不等式的目的是使得缩放后的值还能取得和原式相等的值，重要的是等号能够取到。
### Jensen不等式
对于随机变量的Jensen不等式，当函数$f(x)$是凸函数的时候可以用下式表示：
$$f(E(x)) \le E(f(x))$$
当$f(x)$是凹函数的时候，有
$$f(E(x)) \ge E(f(x))$$

接下来我们就要求解使得式子(6)中杰森不等式等号成立的$q$分布的取值。这里有两种方法可以求解。
### 拉格朗日乘子法
令
$$L(q,\theta) = \sum_z q(z|x_i)log{\frac{p(x_i,z|\theta)}{q(z|x_i)}}, s.t.\sum q(z|x_i) = 1 \tag{7}$$
构建拉格朗日目标函数：
\begin{align\*}
L &= L(q, \theta) + \lambda(\sum_zq(z|x)- 1) \tag{8}\\
&= \sum_z q(z|x_i)log{\frac{p(x_i,z|\theta)}{q(z|x_i)}} + \lambda(\sum_z q(z|x_i) - 1)  \tag{9}
\end{align\*}

对$L$求导，得到：
$$\frac{\partial L}{\partial q(z|x_i)} = log\frac{p(x_i, z|\theta)}{q(z|x_i)} + q(z|x_i)(-\frac{1}{q(z|x_i)}) + \lambda \tag{10}$$
令$\frac{\partial L}{\partial q(z|x_i)}$等于$0$，得到：$$log\frac{p(x_i, z|\theta)}{q(z|x_i)} = 1 - \lambda$$
两边同取$e$的对数：
$$\frac{p(x_i, z|\theta)}{q(z|x_i)} = e^{1-\lambda} \tag{11}$$
$$q(z|x_i) = e^{\lambda - 1}p(x_i, z|\theta) \tag{12}$$
两边同时求和得：
$$1 = e^{\lambda - 1}\sum_z p(x_i, z|\theta) \tag{13}$$
用$p$表示$e^{\lambda-1}$得到：
$$e^{\lambda-1} = \frac{1}{\sum_z p(x_i, z|\theta)}$$
将其代入式子(12)得：
\begin{align\*}
q(z|x_i) &= \frac{p(x_i, z|\theta)}{\sum_z p(x_i, z|\theta)}\\ 
&= \frac{p(z, x_i|\theta)}{p(x_i|\theta)}\\ 
&= p(z|x_i, \theta)  \tag{14}
\end{align\*}

最后求出来$q(z|x_i) = p(z|x_i, \theta)$。

### 杰森不等式成立条件
杰森不等式成立条件是常数，即：
$$\frac{p(x_i, z|\theta)}{q(z|x_i)} = c,  s.t. \sum q(z|x_i)=1 \tag{15}$$
则有:
$$p(x, z_i|\theta) = cq(z_i|x) \tag{16}$$
同时对式子左右两边求和，得到：
$$\sum p(x_i, z|\theta) = \sum cq(z|x_i) = c \tag{17}$$
将$c = \sum p(x_i, z|\theta)$代入式子(14)得：
\begin{align\*}
q(z|x_i) &= \frac{p(x_i, z|\theta)}{\sum p(x_i,z|\theta)}\\
&= \frac{p(x_i, z)|\theta}{p(x_i|\theta)}\\
&= p(z|x_i, \theta) \tag{18}
\end{align\*}

### 等号成立证明
上面两个方法都算出来在$q(z|x_i) = p(z|x_i, \theta)$时$L$能取得最大值。接下来证明这个这个$L$的最大值和$l$相等。
将$q = p(z|x_i, \theta)$代入$L(q, \theta)$得：
\begin{align\*}
L(q, \theta) &= L(p(z|x_i, \theta^t), \theta^t)\\
&= \sum_z p(z|x_i, \theta^t) log\frac{p(z, x_i|\theta^t)}{p(z|x_i, \theta^t)} \\
&= \sum_z p(z|x_i, \theta^t) log p(x_i|\theta^t)\\
&= 1\cdot log p(x_i|\theta^t)\\
&= log p(x_i|\theta^t)\\
&= l(\theta^t; x_i)
\end{align\*}


### 另一种等号成立推导
\begin{align\*}
l(\theta; x) - L(q, \theta) &= l(\theta; x_i) - \sum_z q(z|x_i) log{\frac{p(z, x_i|\theta)}{q(z|x_i)}}\\
&= \sum_z q(z|x_i) log p(x_i|\theta) - \sum_z q(z|x_i) log{\frac{p(z, x_i|\theta)}{q(z|x_i)}}\\
&= \sum_z q(z|x_i)log {\frac{p(x_i|\theta)q(z|x_i)}{p(z, x_i|\theta)}}\\
&= \sum_z q(z|x_i)log {\frac{q(z|x_i)}{p(z|x_i, \theta)}}\\
&= KL(q(z|x_i)||p(z|x_i,\theta))
\end{align\*}
最后算出来两个函数之差是一个KL散度，是从$p$到$q$的KL散度。当前仅当$p=q$时取等，否则就非负。

### M步
\begin{align\*}
L(q, \theta) & = \sum_z q(z|x_i) log\frac{p(z, x_i|\theta)}{q(z|x_i)} \\
& = \underbrace{\sum_z q(z|x_i)log{p(z, x_i|\theta)}}_{Expected complete log-likelyhood} - \underbrace{\sum_z q(z|x_i)l{q(z|x_i)}}_{Entropy}
\end{align\*}

## EM流程
### 计算流程
（１）首先随机初始化模型的不同隐变量对应的参数，
（２）对于每一个观测，首先判断它对应的隐变量的分布。
（３）求期望
（４）最大似然估计求参数
用公式来表示如下：
E步：$q^{t+1} = arg\ max_q L(q, \theta^t)$
M步：$\theta^{t+1} = arg max_{\theta}L(q^{t+1}, \theta)$
E步就是根据$t$时刻的$\theta^t$利用概率$q$求出$L$的期望，然后M步使用最大似然估计计算出新的$\theta$，就这样迭代下去。

## EM收敛性分析
EM算法的收敛性就是要证明$L(q=p(z|x_i, \theta^t) , \theta)$的值一直在增大。
\begin{align\*}
L(p(z|x_i, \theta^{t+1}) , \theta^{t+1}) - L(p(z|x_i, \theta^{t}) , \theta^{t}) &= log p(x_i|\theta^{t+1}) - log p(x_i|\theta^t)\\
& \ge 0
\end{align\*}

## 例子
假如有两个硬币A和B，假设随机从A,B中选一个硬币，掷$10$次，重复$5$次实验，分别求出两个硬币正面向上的概率。假设硬币服从二项分布
$5$次实验结果如下：
5H5T
9H1T
8H2T
4H6T
7H3T

这个时候有两种情况
### 知道每次选的是A还是B
这个时候就变成了极大似然估计。

### 不知道每次选的是A还是B
这个时候就用EM算法了。
首先随机初始化$\theta_A = 0.5, \theta_B = 0.5$，
对于每一个观测，首先判断它对应的隐变量的分布。
$i=\{1,2,3,4,5\}$，分别代表$5$个实验。
首先求出$\theta_A$的参数。
$P(z = A|x_i, \theta_A, \theta_B) = \frac{P(z = A|x_i, \theta_A)}{P(z = A|x_i, \theta_A) + P(z = B|x_i, \theta_B)}$
$P(z = B|x_i, \theta_A, \theta_B) = 1 - P(z = A|x_i,\theta_A,\theta_B)$
然后计算下式：
\begin{align\*}
L(q,\theta_A) &= \sum_{i=1}^5 \sum_zp(z|x_i, \theta_A, \theta_B)log p(x_i|\theta)\\
&= \sum_{i=1}^5 (p(z=A|x_i, \theta_A)log p(x_i|\theta_A) + p(z=B|x_i, \theta_B)log p(x_i|\theta_B))
\end{align\*}
然后利用极大既然估计计算$\theta_A$和$\theta_B$的值。

## 参考文献
1.https://www.zhihu.com/question/27976634/answer/153567695
2.https://en.wikipedia.org/wiki/Jensen%27s_inequality
