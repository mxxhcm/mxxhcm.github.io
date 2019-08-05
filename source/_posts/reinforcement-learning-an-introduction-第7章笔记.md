---
title: reinforcement learning an introduction 第7章笔记
date: 2019-07-30 09:59:50
tags:
 - n-steps
 - 强化学习
categories: 强化学习
mathjax: true
---


## n-step Bootstrapping
这章要介绍的n-step TD方法，将MC和one-step TD用一个统一的框架整合了起来。在一个任务中，两种方法可以无缝切换。n-step方法生成了一系列方法，MC在一头，one-step在另一头。最好的方法往往不是MC也不是one-step TD。One-step TD每隔一个timestep进行bootstrapping，在许多需要及时更新变化的问题，这种方法很有用，但是一般情况下，经历了长时间的稳定变化之后，bootstrap的效果会更好。N-step TD就是将one-step TD方法中time interval的one改成了n。N-step方法的idea和eligibility traces很像，eligibility traces同时使用多个不同的time intervals进行bootstarp。

## n-step TD Prediction
对于使用采样进行policy evaluation的方法来说，不断使用policy $\pi$生成sample episodes，然后估计$v_{\pi}$。MC方法用的是每一个episode中每个state的return进行更新，即无穷步reward之和。TD方法使用每一个state执行完某一个action之后的下一个reward加上下一个state的估计值进行更新。N-step方法使用的是每一个state接下来n步的reward之和加上第n步之后的state的估计值。相应的backup diagram如下所示：
![n_step_diagram](n_step_backup_diagram.png)

n-step方法还是属于TD方法，因为n-step的更新还是基于时间维度上不同estimate的估计进行的。
> n-step updates are still TD methods because they still chagne an erlier estimate based on how it differs from a later estimate. Now the later estimate is not one step later, but n steps later。

正式的，假设$S_t$的estimated value是基于state-reward sequence $S_t, R_{t+1}, S_{t+1}, R_{t+2}, \cdots, R_T,S_T$得到的。
MC方法更新基于completer return：
$$G_T = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots + \gamma^{T-t-1}R_T \tag{1}$$
one-step TD更新基于one-step return：
$$G_{t:t+1} = R_{t+1}+ \gamma V_t(S_{t+1}) \tag{2}$$
two-step TD更新基于two-step return：
$$G_{t:t+2} = R_{t+1}+ \gamma V_t(S_{t+1}) + \gamma^2 V_{t+1}(S_{t+2}) \tag{3}$$
n-step TD更新基于n-step return：
$$G_{t:t+n} = R_{t+1}+ \gamma V_t(S_{t+1}) + \gamma^2 V_{t+1}(S_{t+2}) +\cdots + \gamma^{n-1} R_{t+n} + \gamma^n V_{t+n-1}(S_{t+n}), n\ge1, 0\le t\le T-n \tag{4}$$
所有的n-step方法都可以看成使用n steps的rewards之和加上$V_{t+n-1}(S_{t+n})$近似其余项的rewards近似return。如果$t+n \ge T$时，$T$步以后的reward以及estimated value当做$0$，相当于定义$t+n \ge T$时，$G_{t:t+1} = G_t$。
当$n > 1$时，只有在$t+n$时刻之后，$R_{t+n},S_{t+n}$也是已知的，所以不能使用real time的算法。使用$t+n-1$时刻的$V$近似估计$V_{t+n-1}(S_{t+n})$，将n-step return当做n-step TD的target，得到的更新公式如下：
$$V_{t+n}(S_t) = V_{t+n-1} (S_t) + \gamma(G_{t:t+n} - V_{t+n-1}(S_{t}))\tag{5}$$
当更新$V_{t+n}(S_t)$时，所有其他states的$V$不变，用公式来表示是$V_{t+n}(s) = V_{t+n-1}(s), \forall s\neq S_t$。在每个episode的前$n-1$步中，不进行任何更新，为了弥补这几步，在每个episode结束以后，即到达terminal state之后，仍然继续进行更新，直到下一个episode开始之前，依然进行updates。完整的n-step TD算法如下：

### n-step TD prediction伪代码
n-step TD估计$V\approx v_{\pi}$
输入：一个policy $\pi$
算法参数：step size $\alpha \in (0, 1]$，正整数$n$
随机初始化$V(s), \forall s\in S$
Loop for each episode
$\qquad$初始化 $S_0 \neq terminal$
$\qquad$$T\leftarrow \infty$
$\qquad$Loop for $t=0, 1, 2, \cdots:$
$\qquad$ IF $t\lt T$, THEN
$\qquad\qquad$ 根据$\pi(\cdot|S_t)$执行action
$\qquad\qquad$ 接收并记录$R_{t+1}, S_{t+1}$
$\qquad\qquad$ 如果$S_{t+1}$是terminal ，更新$T\leftarrow t+1$
$\qquad$ END IF
$\qquad$ $\tau \leftarrow t - n + 1, \tau$是当前更新的时间
$\qquad$ If $\tau \ge 0$, then
$\qquad\qquad$ $G\leftarrow \sum_{i=\tau+1}^{min(\tau+n, T)} \gamma^{i-\tau -1} R_i$
$\qquad\qquad$ 如果$\tau+n \lt T$，那么$G\leftarrow G+ \gamma^n V(S_{\tau+n})$
$\qquad\qquad$ $V(S_{\tau}) \leftarrow V(S_{\tau}) +\alpha [G-V(S_{\tau})]$
$\qquad$ End if 
Until $\tau = T-1$
n-step return有一个重要的属性叫做error reduction property，在最坏的情况下，n-step returns的期望也是一个比$V_{t+n-1}$更好的估计：
$$max_{s}|\mathbb{E}\_{\pi}\left[G_{t:t+n}|S_t = s\right]- v_{\pi}(s)| \le \gamma^n max_s | V_{t+n-1}(s)-v_{\pi}(s)| \tag{6}$$
所以所有的n-step TD方法都可以收敛到真实值，MC和one-step TD是其中的一种特殊情况。


## n-step Sarsa
介绍完了prediction算法，接下来要介绍的就是control算法了。因为TD方法是model-free的，所以，还是要估计action value function。上一章介绍了one-step Sarsa，这一章自然介绍一下n-step Sarsa。n-step Sarsa的backup图如下所示：
![n_step_sarsa](n_step_sarsa.png)
就是将n-step TD的state换成state-action就行了。定义action value的n-step returns如下：
$$G_{t:t+n} = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+1} + \cdots + \gamma^{n-1} R_{t+n} + \gamma^n Q_{t+n-1}(S_{t+n},A_{t+n}), n\ge 1, 0\le t\le T-n\tag{7}$$
如果$t+n\ge T$，那么$G_{t:t+n} = G_t$。
完整的$n$-step Sarsa如下所示：

### n-step Sarsa算法伪代码
n-step Sarsa算法，估计$Q\approx q_{\*}$
随机初始化$Q(s,a),\forall s\in S, a\in A$
初始化$\pi$是相对于$Q$的$\epsilon$-greedy policy，或者是一个给定的不变policy
算法参数：step size $\alpha \in (0,1\], \epsilon \gt 0$，一个正整数$n$
Loop for each episode
$\qquad$初始化$S_0\neq$ terminal
$\qquad$ 选择action $A_0= \pi(\cdot| S_0)$
$\qquad$ $T\leftarrow \infty$
$\qquad$ Loop for $t=0,1,2,\cdots$
$\qquad\qquad$ If $t\le T$,then:
$\qquad\qquad\qquad$ 采取action $A_t$，
$\qquad\qquad\qquad$ 接收rewared $R_{t+1}$以及下一个state $S_{t+1}$
$\qquad\qquad$ 如果$S_{t+1}$是terminal，那么
$\qquad\qquad$ $T\leftarrow t+1$
$\qquad\qquad$ 否则选择$A_{t+1} = \pi(\cdot|S_{t+1})$
$\qquad\qquad$End if
$\qquad\qquad$ $\tau \leftarrow t-n+1$
$\qquad\qquad$ If $\tau \ge 0$
$\qquad\qquad\qquad$ $G\leftarrow \sum_{i=\tau +1}^{min(\tau+n, T)} \gamma^{i-\tau -1} R_i$
$\qquad\qquad\qquad$ If $\tau+n \le T$,then
$\qquad\qquad\qquad$ $G\leftarrow G+\gamma^n Q(S_{\tau+n}, A_{\tau+n})$
$\qquad\qquad\qquad$ $Q(S_{\tau}, Q_{\tau}) \leftarrow Q(S_{\tau}, Q_{\tau}) + \alpha \left[G-Q(S_{\tau}, Q_{\tau})\right]$
$\qquad\qquad$End if
$\qquad$ Until $\tau =T-1$

### n-step Expected Sarsa
n-step Expected Sarsa的n-step return定义为：
$$G_{t:t+n} = R_{t+1} +\gamma R_{t+1} + \cdots +\gamma^{n-1} R_{t+n} + \gamma^n \bar{V}\_{t+n-1}(S_{t+n}) \tag{8}$$
其中$\bar{V}_t(s) = \sum_{a}\pi(a|s) Q_t(s,a), \forall s\in S$
如果$s$是terminal，它的期望是$0$。

## n-step Off-policy Learning
Off-policy算法使用behaviour policy采样的内容得到target policy的value function，但是需要使用他们在不同policy下采取某个action的relavtive probability。在$n$-step方法中，我们感兴趣的只有对应的$n$个actions，所以最简单的off-policy $n$-step TD算法，$t$时刻的更新可以使用importance sampling ratio $\rho_{t:t+n-1}$：
$$V_{t+n}(S_t) = V_{t+n-1} S_t + \alpha \rho_{t:t+n-1}\left[G_{t:t+n} - V_{t+n-1}(S_t)\right], 0\le t\le T \tag{9}$$
$\rho_{t:t+n-}$计算的是从$A_t$到$A_{t+n-1}$这$n$个action在behaviour policy和target policy下的relative probability，计算公式如下：
$$\rho_{t:h} = \prod_{k=t}^{min(h, T-1)} \frac{\pi(A_k|S_k)}{b(A_k|S_k)} \tag{10}$$
如果$\pi(A_k|S_k) = 0$，那么对应的$n$-step return对应的权重就应该是$0$，如果policy $pi$下选中某个action的概率很大，那么对应的return就应该比$b$下的权重大一些。因为在$b$下出现的概率下，很少出现，所以权重就该大一些啊。如果是on-policy的话，importance sampling ratio一直是$1$，所以公式$9$可以将on-policy和off-policy的$n$-step TD概括起来。同样的，$n$-step的Sarsa的更新公式也可以换成：
$$Q_{t+n}(S_t,A_t) = Q_{t+n-1}(S_t,A_t) + \alpha \rho_{t+1:t+n}\left[G_{t:t+n} - Q_{t+n-1} (S_t,A_t)\right], 0\le t\le T \tag{11}$$
公式$11$中的importance sampling ratio要比公式$9$中计算的晚一步。这是因为我们是在更新一个state-action pair，我们并不关心有多大的概率选中这个action，我们现在已经选中了它，importance sampling只是用于后续actions的选择。这个解释也让我理解了为什么Q-learning和Sarsa为什么没有使用importance sampling。完整的伪代码如下。
### Off-policy $n$-step Sarsa 估计$Q$
输入：任意的behaviour policy $b$, $b(a|s)\gt 0, \forall s\in S, a\in A$
随机初始化$Q(s,a), \forall s\in S, a\in A$
初始化$\pi$是相对于$Q$的greedy policy
算法参数：步长$\alpha \in (0,1\]$，正整数$n$
Loop for each episode
$\qquad$初始化$S_0\neq terminal$
$\qquad$选择并存储$A_0 \sim b(\cdot|S_0)$
$\qquad$$T\leftarrow \infty$
$\qquad$Loop for $t=0,1,2,\cdots$
$\qquad\qquad$IF $t\lt T$,then
$\qquad\qquad\qquad$执行action $A_t$，接收$R_{t+1}, S_{t+1}$
$\qquad\qquad\qquad$如果$S_{t+1}$是terminal，那么$T\leftarrow t+1$
$\qquad\qquad\qquad$否则选择并记录$A_{t+1} \sim b(\cdot| S_{t+1})$
$\qquad\qquad$END IF
$\qquad\qquad$$\tau \leftarrow t-n +1$  (加$1$表示下标是从$0$开始的)
$\qquad\qquad$IF $\tau \ge 0$
$\qquad\qquad\qquad$$\rho \leftarrow \prod_{i=\tau+1}^{min(\tau+n-1,T-1)} \frac{\pi(A_i|S_i)}{b(A_i|S_i)}$ （计算$\rho_{\tau+1:\tau+n-1}$，这里是不是写成了Expected Sarsa公式）
$\qquad\qquad\qquad$$G\leftarrow \sum_{i=\tau+1}^{min(\tau+n, T)}\gamma^{i-\tau -1}R_i$ （计算$n$个reward, $R_{\tau+1}+\cdots+R_{\tau+n}$）
$\qquad\qquad\qquad$如果$\tau+n \lt T$，$G\leftarrow G + \gamma^n Q(S_{\tau+n},A_{\tau+n})$ （因为没有$Q(S_T,A_T)$）
$\qquad\qquad\qquad$$Q(S_{\tau}, A_{\tau}) \leftarrow Q(S_{\tau}, A_{\tau})+\alpha \rho \left[G-Q(S_{\tau}, A_{\tau})\right]$（计算$Q(S_{\tau+n},A_{\tau+n})$）
$\qquad\qquad\qquad$确保$\pi$是相对于$Q$的greedy policy
$\qquad\qquad$ END IF
$\qquad$Until $\tau = T-1$
上面介绍的算法是$n$-step的Sarsa，计算importance ratio使用的$\rho_{t+1:t+n}$，$n$-step的Expected Sarsa计算importance ratio使用的是$\rho_{t+1:t+n-1}$，因为在估计$\bar{V}_{t+n-1}$时候，考虑到了last state中所有的actions。


## 

## 参考文献
1.《reinforcement learning an introduction》第二版
