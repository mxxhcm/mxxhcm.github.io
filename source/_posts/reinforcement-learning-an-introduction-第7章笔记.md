---
title: reinforcement learning an introduction 第7章笔记
date: 2019-07-30 09:59:50
tags:
 - 强化学习
categories: 强化学习
mathjax: true
---


## n-step Bootstrapping
这章要介绍的n-step TD方法，将MC和one-step TD用一个统一的框架整合了起来。在一个任务中，两种方法可以无缝切换。n-step方法生成了一系列方法，MC在一头，one-step在另一头。最好的方法往往不是MC也不是one-step TD。One-step TD每隔一个timestep进行bootstrapping，在许多需要及时更新变化的问题，这种方法很有用，但是一般情况下，经历了长时间的稳定变化之后，bootstrap的效果会更好。N-step TD就是将one-step TD方法中time interval的one改成了n。N-step方法的idea和eligibility traces很像，eligibility traces同时使用多个不同的time intervals进行bootstarp。

## n-step TD Prediction
对于使用采样进行policy evaluation的方法来说，不断使用policy $\pi$生成sample episodes，然后估计$v\_{\pi}$。MC方法用的是每一个episode中每个state的return进行更新，即无穷步reward之和。TD方法使用每一个state执行完某一个action之后的下一个reward加上下一个state的估计值进行更新。N-step方法使用的是每一个state接下来n步的reward之和加上第n步之后的state的估计值。相应的backup diagram如下所示：
![n\_step\_diagram](n\_step\_backup\_diagram.png)

n-step方法还是属于TD方法，因为n-step的更新还是基于时间维度上不同estimate的估计进行的。
> n-step updates are still TD methods because they still chagne an erlier estimate based on how it differs from a later estimate. Now the later estimate is not one step later, but n steps later。

正式的，假设$S\_t$的estimated value是基于state-reward sequence $S\_t, R\_{t+1}, S\_{t+1}, R\_{t+2}, \cdots, R\_T,S\_T$得到的。
MC方法更新基于completer return：
$$G\_T = R\_{t+1} + \gamma R\_{t+2} + \gamma^2 R\_{t+3} + \cdots + \gamma^{T-t-1}R\_T \tag{1}$$
one-step TD更新基于one-step return：
$$G\_{t:t+1} = R\_{t+1}+ \gamma V\_t(S\_{t+1}) \tag{2}$$
two-step TD更新基于two-step return：
$$G\_{t:t+2} = R\_{t+1}+ \gamma V\_t(S\_{t+1}) + \gamma^2 V\_{t+1}(S\_{t+2}) \tag{3}$$
n-step TD更新基于n-step return：
$$G\_{t:t+n} = R\_{t+1}+ \gamma V\_t(S\_{t+1}) + \gamma^2 V\_{t+1}(S\_{t+2}) +\cdots + \gamma^{n-1} R\_{t+n} + \gamma^n V\_{t+n-1}(S\_{t+n}), n\ge1, 0\le t\le T-n \tag{4}$$
所有的n-step方法都可以看成使用n steps的rewards之和加上$V\_{t+n-1}(S\_{t+n})$近似其余项的rewards近似return。如果$t+n \ge T$时，$T$步以后的reward以及estimated value当做$0$，相当于定义$t+n \ge T$时，$G\_{t:t+1} = G\_t$。
当$n > 1$时，只有在$t+n$时刻之后，$R\_{t+n},S\_{t+n}$也是已知的，所以不能使用real time的算法。使用$t+n-1$时刻的$V$近似估计$V\_{t+n-1}(S\_{t+n})$，将n-step return当做n-step TD的target，得到的更新公式如下：
$$V\_{t+n}(S\_t) = V\_{t+n-1} (S\_t) + \gamma(G\_{t:t+n} - V\_{t+n-1}(S\_{t}))\tag{5}$$
当更新$V\_{t+n}(S\_t)$时，所有其他states的$V$不变，用公式来表示是$V\_{t+n}(s) = V\_{t+n-1}(s), \forall s\neq S\_t$。在每个episode的前$n-1$步中，不进行任何更新，为了弥补这几步，在每个episode结束以后，即到达terminal state之后，仍然继续进行更新，直到下一个episode开始之前，依然进行updates。完整的n-step TD算法如下：

### n-step TD prediction伪代码
n-step TD估计$V\approx v\_{\pi}$
输入：一个policy $\pi$
算法参数：step size $\alpha \in (0, 1]$，正整数$n$
随机初始化$V(s), \forall s\in S$
Loop for each episode
$\qquad$初始化 $S\_0 \neq terminal$
$\qquad$$T\leftarrow \infty$
$\qquad$Loop for $t=0, 1, 2, \cdots:$
$\qquad$ IF $t\lt T$, THEN
$\qquad\qquad$ 根据$\pi(\cdot|S\_t)$执行action
$\qquad\qquad$ 接收并记录$R\_{t+1}, S\_{t+1}$
$\qquad\qquad$ 如果$S\_{t+1}$是terminal ，更新$T\leftarrow t+1$
$\qquad$ END IF
$\qquad$ $\tau \leftarrow t - n + 1, \tau$是当前更新的时间
$\qquad$ If $\tau \ge 0$, then
$\qquad\qquad$ $G\leftarrow \sum\_{i=\tau+1}^{min(\tau+n, T)} \gamma^{i-\tau -1} R\_i$
$\qquad\qquad$ 如果$\tau+n \lt T$，那么$G\leftarrow G+ \gamma^n V(S\_{\tau+n})$
$\qquad\qquad$ $V(S\_{\tau}) \leftarrow V(S\_{\tau}) +\alpha [G-V(S\_{\tau})]$
$\qquad$ End if 
Until $\tau = T-1$
n-step return有一个重要的属性叫做error reduction property，在最坏的情况下，n-step returns的期望也是一个比$V\_{t+n-1}$更好的估计：
$$max\_{s}|\mathbb{E}\_{\pi}\left[G\_{t:t+n}|S\_t = s\right]- v\_{\pi}(s)| \le \gamma^n max\_s | V\_{t+n-1}(s)-v\_{\pi}(s)| \tag{6}$$
所以所有的n-step TD方法都可以收敛到真实值，MC和one-step TD是其中的一种特殊情况。


## n-step Sarsa
介绍完了prediction算法，接下来要介绍的就是control算法了。因为TD方法是model-free的，所以，还是要估计action value function。上一章介绍了one-step Sarsa，这一章自然介绍一下n-step Sarsa。n-step Sarsa的backup图如下所示：
![n\_step\_sarsa](n\_step\_sarsa.png)
就是将n-step TD的state换成state-action就行了。定义action value的n-step returns如下：
$$G\_{t:t+n} = R\_{t+1} + \gamma R\_{t+2} + \gamma^2 R\_{t+1} + \cdots + \gamma^{n-1} R\_{t+n} + \gamma^n Q\_{t+n-1}(S\_{t+n},A\_{t+n}), n\ge 1, 0\le t\le T-n\tag{7}$$
如果$t+n\ge T$，那么$G\_{t:t+n} = G\_t$。
完整的$n$-step Sarsa如下所示：

### n-step Sarsa算法伪代码
n-step Sarsa算法，估计$Q\approx q\_{\*}$
随机初始化$Q(s,a),\forall s\in S, a\in A$
初始化$\pi$是相对于$Q$的$\epsilon$-greedy policy，或者是一个给定的不变policy
算法参数：step size $\alpha \in (0,1\], \epsilon \gt 0$，一个正整数$n$
Loop for each episode
$\qquad$初始化$S\_0\neq$ terminal
$\qquad$ 选择action $A\_0= \pi(\cdot| S\_0)$
$\qquad$ $T\leftarrow \infty$
$\qquad$ Loop for $t=0,1,2,\cdots$
$\qquad\qquad$ If $t\lt T$,then:
$\qquad\qquad\qquad$ 采取action $A\_t$，
$\qquad\qquad\qquad$ 接收rewared $R\_{t+1}$以及下一个state $S\_{t+1}$
$\qquad\qquad$ 如果$S\_{t+1}$是terminal，那么
$\qquad\qquad$ $T\leftarrow t+1$
$\qquad\qquad$ 否则选择$A\_{t+1} = \pi(\cdot|S\_{t+1})$
$\qquad\qquad$End if
$\qquad\qquad$ $\tau \leftarrow t-n+1$
$\qquad\qquad$ If $\tau \ge 0$
$\qquad\qquad\qquad$ $G\leftarrow \sum\_{i=\tau +1}^{min(\tau+n, T)} \gamma^{i-\tau -1} R\_i$
$\qquad\qquad\qquad$ If $\tau+n \le T$,then
$\qquad\qquad\qquad$ $G\leftarrow G+\gamma^n Q(S\_{\tau+n}, A\_{\tau+n})$
$\qquad\qquad\qquad$ $Q(S\_{\tau}, Q\_{\tau}) \leftarrow Q(S\_{\tau}, Q\_{\tau}) + \alpha \left[G-Q(S\_{\tau}, Q\_{\tau})\right]$
$\qquad\qquad$End if
$\qquad$ Until $\tau =T-1$

### n-step Expected Sarsa
n-step Expected Sarsa的n-step return定义为：
$$G\_{t:t+n} = R\_{t+1} +\gamma R\_{t+1} + \cdots +\gamma^{n-1} R\_{t+n} + \gamma^n \bar{V}\_{t+n-1}(S\_{t+n}) \tag{8}$$
其中$\bar{V}\_t(s) = \sum\_{a}\pi(a|s) Q\_t(s,a), \forall s\in S$
如果$s$是terminal，它的期望是$0$。

## n-step Off-policy Learning
Off-policy算法使用behaviour policy采样的内容得到target policy的value function，但是需要使用他们在不同policy下采取某个action的relavtive probability。在$n$-step方法中，我们感兴趣的只有对应的$n$个actions，所以最简单的off-policy $n$-step TD算法，$t$时刻的更新可以使用importance sampling ratio $\rho\_{t:t+n-1}$：
$$V\_{t+n}(S\_t) = V\_{t+n-1} S\_t + \alpha \rho\_{t:t+n-1}\left[G\_{t:t+n} - V\_{t+n-1}(S\_t)\right], 0\le t\le T \tag{9}$$
$\rho\_{t:t+n-}$计算的是从$A\_t$到$A\_{t+n-1}$这$n$个action在behaviour policy和target policy下的relative probability，计算公式如下：
$$\rho\_{t:h} = \prod\_{k=t}^{min(h, T-1)} \frac{\pi(A\_k|S\_k)}{b(A\_k|S\_k)} \tag{10}$$
如果$\pi(A\_k|S\_k) = 0$，那么对应的$n$-step return对应的权重就应该是$0$，如果policy $pi$下选中某个action的概率很大，那么对应的return就应该比$b$下的权重大一些。因为在$b$下出现的概率下，很少出现，所以权重就该大一些啊。如果是on-policy的话，importance sampling ratio一直是$1$，所以公式$9$可以将on-policy和off-policy的$n$-step TD概括起来。同样的，$n$-step的Sarsa的更新公式也可以换成：
$$Q\_{t+n}(S\_t,A\_t) = Q\_{t+n-1}(S\_t,A\_t) + \alpha \rho\_{t+1:t+n}\left[G\_{t:t+n} - Q\_{t+n-1} (S\_t,A\_t)\right], 0\le t\le T \tag{11}$$
公式$11$中的importance sampling ratio要比公式$9$中计算的晚一步。这是因为我们是在更新一个state-action pair，我们并不关心有多大的概率选中这个action，我们现在已经选中了它，importance sampling只是用于后续actions的选择。这个解释也让我理解了为什么Q-learning和Sarsa为什么没有使用importance sampling。完整的伪代码如下。
### Off-policy $n$-step Sarsa 估计$Q$
输入：任意的behaviour policy $b$, $b(a|s)\gt 0, \forall s\in S, a\in A$
随机初始化$Q(s,a), \forall s\in S, a\in A$
初始化$\pi$是相对于$Q$的greedy policy
算法参数：步长$\alpha \in (0,1\]$，正整数$n$
Loop for each episode
$\qquad$初始化$S\_0\neq terminal$
$\qquad$选择并存储$A\_0 \sim b(\cdot|S\_0)$
$\qquad T\leftarrow \infty$
$\qquad$Loop for $t=0,1,2,\cdots$
$\qquad\qquad$IF $t\lt T$,then
$\qquad\qquad\qquad$执行action $A\_t$，接收$R\_{t+1}, S\_{t+1}$
$\qquad\qquad\qquad$如果$S\_{t+1}$是terminal，那么$T\leftarrow t+1$
$\qquad\qquad\qquad$否则选择并记录$A\_{t+1} \sim b(\cdot| S\_{t+1})$
$\qquad\qquad$END IF
$\qquad\qquad \tau \leftarrow t-n +1$  (加$1$表示下标是从$0$开始的)
$\qquad\qquad$ IF $\tau \ge 0$
$\qquad\qquad\qquad \rho \leftarrow \prod\_{i=\tau+1}^{min(\tau+n,T)} \frac{\pi(A\_i|S\_i)}{b(A\_i|S\_i)}$ （计算$\rho\_{\tau+1:\tau+n}$，这里是不是写成了Expected Sarsa公式）
$\qquad\qquad\qquad G\leftarrow \sum\_{i=\tau+1}^{min(\tau+n, T)}\gamma^{i-\tau -1}R\_i$ （计算$n$个reward, $R\_{\tau+1}+\cdots+R\_{\tau+n}$）
$\qquad\qquad\qquad$如果$\tau+n \lt T$，$G\leftarrow G + \gamma^n Q(S\_{\tau+n},A\_{\tau+n})$ （因为没有$Q(S\_T,A\_T)$）
$\qquad\qquad\qquad Q(S\_{\tau}, A\_{\tau}) \leftarrow Q(S\_{\tau}, A\_{\tau})+\alpha \rho \left[G-Q(S\_{\tau}, A\_{\tau})\right]$（计算$Q(S\_{\tau+n},A\_{\tau+n})$）
$\qquad\qquad\qquad$确保$\pi$是相对于$Q$的greedy policy
$\qquad\qquad$ END IF
$\qquad$Until $\tau = T-1$
上面介绍的算法是$n$-step的Sarsa，计算importance ratio使用的$\rho\_{t+1:t+n}$，$n$-step的Expected Sarsa计算importance ratio使用的是$\rho\_{t+1:t+n-1}$，因为在估计$\bar{V}\_{t+n-1}$时候，考虑到了last state中所有的actions。


## $n$-step Tree Backup算法
这一章介绍的是不适用importance sampling的off-policy算法，叫做tree-backup algorithm。如下图所示，是一个$3$-step的tree-backup diaqgram。
沿着中间这条路走，有三个sample states和rewards以及两个sample actions。在旁边的是没有被选中的actions。对于这些没有选中的actions，因为没有采样，所以使用bootstrap计算target value。因为它的backup diagram有点像tree，所以就叫做tree backup upadte。或者更精确的说，backup update使用的是所用tree的叶子结点action value的估计值进行计算的。非叶子节点的action对应的是采样的action，不参与更新，所有叶子节点对于target的贡献都有一个权重正比于它在target policy下发生的概率。在$S\_{t+1}$处的除了$A\_{t+1}$之外所有action权重是$\pi(a|S\_{t+1})$，$A\_{t+1}$一点贡献没有；在$S\_{t+2}$处所有没有被选中的action的权重是$\pi(A\_{t+1}|S\_{t+1})\pi(a'|S\_{t+2})$；在$S\_{t+3}$处所有没有被选中的action的权重是$\pi(A\_{t+1}|S\_{t+1})\pi(A\_{t+2}|S\_{t+2})\pi(a''|S\_{t+3})$
one-step Tree backup 算法其实就是Expected Sarsa：
$$G\_{t:t+1} = R\_{t+1} + \gamma \sum\_a\pi(a|S\_{t+1})Q(a, S\_{t+1}), t\lt T-1 \tag{12}$$
two-step Tree backup算法return计算公式如下：
\begin{align\*}
G\_{t:t+2} &= R\_{t+1} + \gamma \sum\_{a\neq A\_{t+1}}\pi(a|S\_{t+1})Q(a, S\_{t+1}), t\lt T-1$ + \gamma\pi(A\_{t+1}|S\_{t+1})(R\_{t+2} + \sum\_{a} \pi(a|S\_{t+2})Q(a, S\_{t+2})\\\\
&=R\_{t+1} + \gamma \sum\_{a\neq A\_{t+1}}\pi(a|S\_{t+1})Q(a, S\_{t+1}), t\lt T-1$ + \gamma\pi(A\_{t+1}|S\_{t+1})G\_{t+1:t+2}, t \lt T-2
\end{align\*}
下面的形式给出了tree-backup的递归形式如下：
$$G\_{t:t+n} = R\_{t+1} + \gamma \sum\_{a\neq A\_{t+1}}\pi(a|S\_{t+1})Q(a,S\_{t+1}) + \gamma\pi(A\_{t+1}|S\_{t+1}) G\_{t+1:t+n}, n\ge 2, t\lt T-1, \tag{13}$$
当$n=1$时除了$G\_{T-1:t+n} = R\_T$，其他的和式子$12$一样。使用这个新的target代替$n$-step Sarsa中的target就可以得到$n$-step tree backup 算法。
完整的算法如下所示：

### $n$-step Tree Backup 算法
随机初始化$Q(s,a),\forall s\in S, a\in A$
初始化$\pi$是相对于$Q$的$\epsilon$-greedy policy，或者是一个给定的不变policy
算法参数：step size $\alpha \in (0,1\], \epsilon \gt 0$，一个正整数$n$
Loop for each episode
$\qquad$初始化$S\_0\neq$ terminal
$\qquad$ 根据$S\_0$随机选择action $A\_0$
$\qquad$ $T\leftarrow \infty$
$\qquad$ Loop for $t=0,1,2,\cdots$
$\qquad\qquad$ If $t\lt T$,then:
$\qquad\qquad\qquad$ 采取action $A\_t$，
$\qquad\qquad\qquad$ 接收rewared $R\_{t+1}$以及下一个state $S\_{t+1}$
$\qquad\qquad$ 如果$S\_{t+1}$是terminal，那么
$\qquad\qquad$ $T\leftarrow t+1$
$\qquad\qquad$ 根据$S\_{t+1}$随机选择action $A\_{t+1}$
$\qquad\qquad$End IF
$\qquad\qquad$ $\tau \leftarrow t-n+1$
$\qquad\qquad\qquad$ IF $\tau+n\ge T$ then
$\qquad\qquad\qquad\qquad G\leftarrow R\_T$
$\qquad\qquad\qquad$ ELSE 
$\qquad\qquad\qquad\qquad G\leftarrow R\_{t+1}+\gamma \sum\_a\pi(a|S\_{t+1})Q(a, S\_{t+1})$ 
$\qquad\qquad\qquad$ END IF 
$\qquad\qquad\qquad$Loop for $k = min(t, T-1)$ down $\tau+1$
$\qquad\qquad\qquad G\leftarrow R\_k+\gamma^n\sum\_{a\neq A\_k}\pi(a|S\_k)Q(a, S\_k) + \gamma \pi(A\_k|S\_k) G$ 
$\qquad\qquad\qquad$ $Q(S\_{\tau}, Q\_{\tau}) \leftarrow Q(S\_{\tau}, Q\_{\tau}) + \alpha \left[G-Q(S\_{\tau}, Q\_{\tau})\right]$
$\qquad\qquad$End if
$\qquad$ Until $\tau =T-1$

## $n$-step $Q(\sigma)$
现在已经讲了$n$-step Sarsa，$n$-step Expected Sarsa，$n$-step Tree Backup，$4$-step的一个backup diagram如下所示。它们其实有很多共同的特性，可以用一个框架把它们统一起来。
具体的算法就不细说了。

## 参考文献
1.《reinforcement learning an introduction》第二版
2.https://stats.stackexchange.com/questions/335396/why-dont-we-use-importance-sampling-for-one-step-q-learning
