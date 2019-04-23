---
title: '熵，交叉熵，相对熵（KL散度），条件熵，互信息'
date: 2018-12-23 10:54:31
tags:
  - 机器学习
  - 熵
  - 交叉熵
  - 条件熵
  - 相对熵
  - 交叉熵
  - KL散度 
  - 互信息
categories: 机器学习
mathjax: true
---

## 乡农熵(Shannon entropy)
乡农定义了一个事件的信息量是其发生概率的负对数($-log(p)$)，即乡农信息量，乡农熵是信息量的期望。

### 介绍
这里的熵都是指的信息论中的熵，也叫乡农熵(shannon entropy)。通常，熵是无序或不确定性的度量。
与每个变量可能的取值相关的信息熵是每个可能取值的概率质量函数的负对数：
$$S = - \sum_i P_i lnP_i$$
当事件发生的概率较低时，该事件比高概率事件携带更多“信息”。这种方式定义的每个事件所携带的信息量是一个随机变量，事实上乡农熵定义的一个事件的信息量就是这个事件发生的概率的负对数，这个随机变量（信息量）的期望值是信息熵。信息熵通常以比特(或者称为shannons),自然单位(nats)或十进制数字(dits，bans或hartleys)来测量。具体的单位取决于用于定义熵的对数的基。
采用概率分布的对数形式作为信息的度量的原因是因为它的可加性。例如，投掷公平硬币的熵是$1$比特，投掷$m$个硬币的熵是$m$比特。以比特为单位的时候，如果$n$是$2$的指数次方，则需要$log_2n$位来表示一个具有$n$个取值的变量。如果该变量的$n$个取值发生的可能性是相等的，则熵等于$log_2n$。
如果一个事件发生的可能性比其他事件发生的可能性更高，观察到该事件发生的信息量少于观测到一些罕见事件，即观测到更罕见的事件时能提供更多的信息。由于小概率事件发生的可能性更低，因此最终的结果是从非均匀分布的数据接收的熵总是小于或等于$log_2n$。当一个结果一定发生时，熵为零。
但是熵仅仅量化考虑事件发生的概率，它封装的信息是有关概率分布的信息，事件本身的意义在这种度量方式的定义中无关紧要。
熵的另一种解释是最短平均编码长度。

### 定义
乡农定义了entropy, 定义离散型随机变量$X$，其可能取值为${x_1,\cdots,x_n}$，它对应的概率质量函数(probability mass function) P(X)，则熵$H$为：
$$H(X) = E[I(X)] = E[-log(P(X))]$$
其中$E$是求期望，$I$是随机变量$X$的信息量, $I(X)$本身是一个随机变量。
它可以显示写成：
$$H(X) = \sum_{i=1}^nP(x_i)I(x_i) = -\sum_{i=1}^nP(x_i)log_bP(x_i)$$
其中b是自然对数的底，$b$常取的值为$2,e,10$，对应的熵的单位是bits, nats，bans。
当$P(x_i)=0$的时候，对应的$PlogP$的值为$0log_b(0)$, 和极限(limit)是一致的：
$$lim_{p\rightarrow 0_+}plog(p) = 0.$$
#### 连续型随机变量的熵
将概率质量函数替换为概率密度函数，即可得到连续性随机变量的熵：
$$h[f] = E[-ln(f(x))] = - \int_X f(x)ln(f(x))dx.$$

### 示例
抛一枚硬币，已知其正反两面出现的概率是不相等的，求其正面朝上的概率，该问题可以看做一个伯努利分布问题。
如果硬币是公平的，此时得到结果的熵是最大的。这是因为此时抛一次抛硬币的结果具有最大的不确定性。每一个抛硬币的结果会占满一整个bit位。因为
\begin{align\*}
H(X) &= - \sum_{i=1}^n P(x_i)log_bP(x_i)\\
&= - \sum_{i=1}^2\frac{1}{2}log_2\frac{1}{2}\\
&= - \sum_{i=1}^2\frac{1}{2}\cdot(-1)\\
&= 1
\end{align\*}
如果硬币是不公平的，正面向上的概率是$p$，反面向上的概率是$q$，$p \ne q$, 则结果的不确定性更小。因为每次抛硬币，出现其中一面的可能性要比另一面要大，减小的不确定性就得到了更小的熵：每一次抛硬币得到的信息都会小于$1$bit，比如，$p=0.7$时：
\begin{align\*}
H(X) &= -plog_2p - qlog_2q\\
&= -0.7log_20.7 - 0.3log_20.3\\
&= -0.7\cdot(-0.515) - 0.3\cdot(-1.737)\\
&= 0.8816\\
&\le 1
\end{align\*}
上面的例子证明不确定性跟变量取值的概率有关。
不确定性也跟变量的取值个数有关，上面例子的极端情况是正反面一样（即只有一种取值），那么熵就是0，没有不确定性。

### 解释(rationale)
为什么乡农定义了信息量为$-log(p)$？$-\sum p_i log(p_i)$的意义是什么？
首先我们需要想一想信息量需要满足什么条件，然后定义一个信息函数I表示发生概率为$p_i$的事件$i$的信息量，那么这个信息函数需要满足以下条件。
- $I(p)$是单调下降的；
- $I(p) \ge 0$, 即信息是非负的；
- $I(1) = 0$, 一定发生的事件不包含信息；
- $I(p_1p_2) = I(p_1) + I(p_2)$, 独立事件包含的信息是可加的。
最后一个条件很关键，它指出了两个独立事件的联合分布和两个分开的独立事件所包含的信息是一样多的。例如，$A$事件有$m$个等可能性的结果，$B$事件有$n$个等可能性的结果，$AB$有$mn$个等可能性的结果。$A$事件需要$log_2(m)$bits去编码，$B$事件需要用$log_2(n)$bits去编码，$AB$需要$log_2(mn) = log_2(m) + log_2(n)$bits编码。乡农发现了$log$函数能够保留可加性，即：
$$I(p) = log(\frac{1}{p}) = - log(p)$$
事实上，这个函数$I$是唯一的(可以证明),选择$I$当做信息函数。如果一个分布中事件$i$发生的概率是$p_i$,那么采样$N$次，事件$i$发生的次数为$n_i = N p_i$, 所有$n_i$次的信息为$$\sum_in_iI(p_i) = - \sum_iNp_ilog(p_i).$$
每个事件的平均信息就是：
$$-\sum_ip_ilog(p_i)$$
所以$-\sum_ip_ilog(p_i)$就是信息量的期望，即信息熵。
在信息论中，熵的另一种解释是最短平均编码长度。

## 交叉熵(cross entropy)
### 介绍
交叉熵用于衡量估计的概率分布与真实概率分布之间的差异。
交叉熵是信息熵的推广。假设有两个分布$p$和$q$，$p$是真实分布，$q$是非真实分布。信息熵是用真实分布$p$来衡量识别一个事件所需要的编码长度的期望。而交叉熵是用非真实分布$q$来估计真实分布$p$的编码长度的期望，用$q$来编码的事件来自分布$p$，所以期望中使用的概率是$p(i)$。$H(p,q)$称为交叉熵。

### 定义
分布p和q在给定集合X的交叉熵定义为：
$$H(p,q) = E_p[-logq] = H(p) + D_{KL}(p||q)$$
其中$H(p)$是$p$的信息熵，$D_{KL}(p||q)$是从$q$到$p$的$K-L$散度，或者说$p$相对于$q$的相对熵。

### 示例 
如含有4个字母$(A,B,C,D)$的数据集中，$p=(\frac{1}{2}, \frac{1}{2}, 0, 0)$，即$A$和$B$出现的概率均为$\frac{1}{2}$，$C$和$D$出现的概率都为$0$。$H(p)$为$1$，即只需要$1$位编码即可识别$A$和$B$。如果使用分布$q=(\frac{1}{4},\frac{1}{4},\frac{1}{4},\frac{1}{4})$来编码则得到$H(p,q)=2$，即需要$2$位编码来识别$A$和$B$。

### 解释
在机器学习中，交叉熵用于衡量估计的概率分布与真实概率分布之间的差异。
在信息论中，Kraft-McMillan定理建立了任何可直接解码的编码方案，为了识别一个$X$的可能值$x_i$f可以看做服从一个在$X$上的隐式概率分布$q(x_i)=2^{-l_i}$,其中$l_i$是$x_i$的编码长度，单位是bits。因此，交叉熵可以被解释为当数据服从真实分布$p$时，在假设分布$q$下得到的每个信息编码长度的期望。

### 性质
- $H(p,q) \ge H(p)$,由吉布森不等式可以知道，该式子恒成立，当$q$等于$p$时等号成立。

### to do ?交叉熵损失函数和logistic regression之间的关系。

## $K-L$散度(Kullback-Leibler divergence)
### 介绍
$K-L$散度也叫相对熵(relative entropy)，是用来衡量估计分布和真实分布之间的差异性。
$K-L$散度也叫相对熵(relative entropy)，信息熵是用来度量信息量的，信息熵给出了最小熵是多少，但是信息熵并没有给出如何得到最小熵，$K-L$散度也没有给出来如何得到最小熵。但是$K-L$散度可以用来衡量用一个带参数的估计分布来近似真实数据分布时损失了多少信息，可以理解为根据非真实分布$q$得到的平均编码长度比由真实分布$p$得到的平均编码长度多出的长度叫做相对熵。

### 定义
#### 离散型随机变量
给定概率分布$P$和$Q$在相同的空间中，它们的$K-L$散度定义为：
\begin{align\*}
D_{KL}(P||Q) &= -\sum_iP(i)(logQ(i)) - (-\sum_iP(i)logP(i))\\ 
D_{KL}(P||Q) &= \sum_iP(i)(logP(i) - logQ(i))\\ 
D_{KL}(P||Q) &= -\sum_iP(i)log(\frac{Q(i)}{Q(i)})\\
D_{KL}(P||Q) &= \sum_iP(i)log(\frac{P(i)}{Q(i)})
\end{align\*}
可以看出，$K-L$散度是概率分布$P$和$Q$对数差相对于概率分布$P$的期望。需要注意的是$D_{KL}(P||Q) \ne D_{KL}(Q||P),$因为$P$和$Q$的地位是不同的。相对熵的前半部分就是交叉熵，后半部分是相对熵。
#### 连续型随机变量
对于连续性随机变量的分布$P$和$Q$，$K-L$散度被定义为积分：
$$D_{KL}(P||Q) = \int_{-\infty}^{infty}p(x)log(\frac{p(x)}{q(x)})dx,$$
其中$p$和$q$代表分布$P$和分布$Q$的概率密度函数。
更一般的，$P$和$Q$表示是同一个集合$X$的概率分布，$P$相对于$Q$是绝对连续的，从$Q$到$P$的$K-L$散度定义为：
$$D_{KL}(P||Q) = \int_X log(\frac{dP}{dQ})dP$$
上式可以被写成：
$$D_{KL}(P||Q) = \int_X log(\frac{dP}{dQ})\frac{dP}{dQ}dP$$
可以看成$\frac{P}{Q}$的熵。

### 示例
$P$是一个二项分布，$P~(2,0.4)$，$Q$是一个离散型均匀分布，$x = 0,1,2$, 每一个取值的概率都是$p=\frac{1}{3}$。

| |0|1|2|
|--|--|--|--|
|$P$分布|0.36|0.48|0.16|
|$Q$分布|0.333|0.333|0.333|
$K-L$散度的计算公式如下（使用自然对数）：
\begin{align\*}
D_{KL}(Q||P) &= \sum_iQ(i)ln(\frac{Q(i)}{P(i)})\\
& = 0.333ln(\frac{0.333}{0.36}) + 0.333ln(\frac{0.333}{0.48}) + 0.333ln(\frac{0.333}{0.16})\\
& = -0.02596 + (-0.12176) + 0.24408\\
& = 0.09637(nats)
\end{align\*}
上面计算出来的是从$P$到$Q$的K-L散度，或者$Q$相对于$P$的相对熵。

### 解释
从$Q$到$P$的$K-L$散度表示为$D_{KL}(P||Q)$。在机器学习中，$D_{KL}(P||Q)$被称为信息增益。
在信息论中，$K-L$散度也被称为$P$相对于$Q$的相对熵。从信息编码角度来看，$D_{KL}(P||Q)$可以看做用估计分布$q$得到的平均编码长度比用真实分布p得到的平均编码长度多出的长度。

### 性质
- 非负性，$D_{KL}(P||Q)\ge 0$,当且仅当$P=Q$时等号成立。
- 可加性，如果$P_1,P_2$的分布是独立的，即$P(x,y) = P_1(x)P_2(y)$, $Q,Q_1,Q_2$类似，那么：
$$D_{KL}(P||Q) = D_{KL}(P_1||Q_1) + D_{KL}(P_2||Q_2)$$
- 不对称性，所以K-L散度不是距离，距离需要满足对称性。

## 条件熵

### 定义
给定$X$，$Y$的条件熵定义如下：
给定离散变量${\displaystyle X}$和${\displaystyle Y}$,给定${\displaystyle X}$以后，${\displaystyle Y}$的条件熵定义为每一个${\displaystyle x}$使用权值${\displaystyle p(x)}$ 的加权和${\displaystyle \mathrm {H} (Y|X=x)}$。
$$H(Y|X) &\equiv \sum_{x\in\bold{X}}p(x)H(Y|X=x)$$
可以证明它等价于下式：
$$H(Y|X) = -\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)log{\frac{p(X,Y)}{p(X)}}$$

### 证明
\begin{align\*}
H(Y|X) &\equiv \sum_{x\in\bold{X}}p(x)H(Y|X=x)\\
&=-\sum_{x\in\bold{X}}p(x)\sum_{y\in \bold{Y}}p(y|x)logp(y|x)\\
&=-\sum_{x\in\bold{X}}\sum_{y\in \bold{Y}}p(x)p(y|x)logp(y|x)\\
&=-\sum_{x\in\bold{X},y\in \bold{Y}}p(x,y)logp(y|x)\\
&=-\sum_{x\in\bold{X},y\in \bold{Y}}p(x,y)\frac{logp(x,y)}{logp(x)}\\
&=\sum_{x\in\bold{X},y\in \bold{Y}}p(x,y)\frac{logp(x)}{logp(x,y)}\\
\end{align\*}
### 属性
- 当且仅当$Y$完全由$X$的值确定时，条件熵为$0$。
- 当且仅当$X$和$Y$是独立随机变量的时候，$H(Y|X) = H(Y)$。
- 链式法则。假设一个系统由随机变量$X,Y$确定，他们有联合熵$H(X,Y)$，我们需要$H(X,Y)$个比特去表述这个系统，如果我们已经知道了$X$的值，相当于我们已经有了$H(X)$位的信息。一旦$X$已知了，我们只需要$H(X,Y)-H(X)$位去描述整个系统。所以就有了链式法则：$H(Y|X) = H(X,Y) - H(X)$。
\begin{align\*}
H(Y|X) &= \sum_{X\in \bold{X}, Y\in \bold{Y}}p(X,Y)log{\frac{p(X)}{p(X,Y)}}\\
&= - \sum_{X\in \bold{X}, Y\in \bold{Y}}p(X,Y)log{p(X,Y)}+\sum_{X\in \bold{X}, Y\in \bold{Y}}p(X,Y)log{p(X)}\\
&=H(X,Y) +\sum_{X\in \bold{X}}p(X)log{p(X)}\\
&=H(X,Y) - H(X)
\end{align\*}
- 贝叶斯公式。$H(Y|X) = H(X|Y) - H(X) + H(Y)$。
证明：$H(Y|X)=H(X,Y) - H(X),H(X|Y) = H(X,Y) - H(Y)$。两个式子相减就可以得到。

## 互信息
决策树中的信息增益指的是互信息不是KL散度。
### 定义
用$(X,Y)$表示空间$\bold{X}\times\bold{Y}$上的一对随机变量，他们的联合分布是$P_{(X,Y)}$，边缘分布是$P_X,P_Y)$，信息熵被定义为：
$I(X;Y) = D_{KL}(P_{(X,Y)}||P_XP_Y)$
对于离散变量：
$I(X;Y)=\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)log(\frac{p(X,Y)}{p(X)p(Y)})$
对于随机变量：
$I(X;Y)=\int_X\int_Y p(X,Y)log(\frac{p(X,Y)}{p(X)p(Y)})dxdy$

## 信息熵，相对熵，交叉熵，条件熵，互信息之间的关系
### 信息论
信息熵是对随机事件用真实的概率分布$p$进行编码的长度的期望，是最短平均编码长度。
交叉熵是对随机事件用估计的概率分布$q$按照其真实概率分布$p$进行编码的长度的期望（随机事件是从真实概率分布$p$中取的，但是用分布$q$进行编码），大于等于最短平均编码长度，只有$q$等于真实分布$q$时，才是最短编码长度。
而相对熵对随机事件用估计的概率分布$q$比用真实的概率分布$p$进行编码多用的编码长度，如果$p$和$q$相等的话，相对熵为$0$。

### 机器学习
在机器学习中，交叉熵通常作为一个loss函数，用来衡量真实分布$p$和估计分布$q$之间的差异。而$K-L$散度也是用来衡量两个概率分布的差异，但是多了一个信息熵项。$K-L$散度的前半部分是交叉熵，后半部分是真实分布$p$的信息熵。(一个我自己认为的不严谨的说法是相对熵算的是相对值，而交叉熵算的是绝对值)。交叉熵正比于负的对数似然估计，最小化交叉熵等价于最大化对数似然估计。
如果$p$是固定的，那么随着$q$的增加相对熵也在增加，但是如果$p$是不固定的，很难说相对熵是差异的绝对量度，因为它随着$p$的增长而改变。而在机器学习领域，真实分布$p$是固定的，随着$q$的改变，$H(p)$是不变的,也就是信息熵是固定的。所以，从优化的角度来说，最小化交叉熵也就是最小化了相对熵。但是在其他领域，$p$可能是变化的，最小化交叉熵和最小化相对熵就不是等价的了。

### 互信息和条件熵，相对熵的关系
互信息可以被等价定义为：
\begin{align\*}
I(X;Y)& \equiv H(X)-H(X|Y)\\
&\equiv H(Y) - H(Y|X)\\
&\equiv H(X)+H(Y)-H(X,Y)\\
&\equiv H(X,Y)-H(X|Y)-H(Y|X)\\
\end{align\*}

证明：
\begin{align\*}
I(X;Y)&=\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)log(\frac{p(Y,Y)}{p(X)p(Y)})\\
&=\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)log(\frac{p(Y,Y)}{p(X)})-\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)logp(Y)\\
&=\sum_{X\in \bold{X},Y\in \bold{Y}}p(X)P(Y|X)logp(Y|X)-\sum_{X\in \bold{X},Y\in \bold{Y}}p(X,Y)logp(Y)\\
&=\sum_{X\in \bold{X}}p(X)(\sum_{Y\in \bold{Y}}P(Y|X)logp(Y|X))-\sum_{Y\in \bold{Y}}(\sum_{X\in \bold{X}}p(X,Y))logp(Y)\\
&=\sum_{X\in \bold{X}}p(X)H(Y|X=x)-\sum_{Y\in \bold{Y}}p(Y)logp(Y)\\
&=-H(Y|X)+H(Y)\\
&=H(Y)-H(Y|X)
\end{align\*}

互信息和KL散度的联系：
从联合分布$p(x,y)$到边缘分布$p(X)p(Y)$或者条件分布$p(X|Y)p(X)$的KL散度。

## 参考文献(references)
1.https://en.wikipedia.org/wiki/Entropy_(information_theory)
2.https://zh.wikipedia.org/wiki/熵_(信息论)
3.https://www.zhihu.com/question/22178202/answer/49929786
4.https://en.wikipedia.org/wiki/Cross_entropy
5.https://en.wikipedia.org/wiki/Kullback-Leibler_divergence
6:https://www.zhihu.com/question/41252833/answer/108777563
7.https://www.zhihu.com/question/41252833/answer/141598211
8.https://stats.stackexchange.com/questions/265966/why-do-we-use-kullback-leibler-divergence-rather-than-cross-entropy-in-the-t-sne
9.https://en.wikipedia.org/wiki/Conditional_entropy
10.https://en.wikipedia.org/wiki/Mutual_information
11.https://zhuanlan.zhihu.com/p/26551798
12.https://blog.csdn.net/gangyin5071/article/details/82228827#4相对熵kl散度
