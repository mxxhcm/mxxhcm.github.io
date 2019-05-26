---
title: reinforcement learning an introduction 第5章笔记
date: 2019-04-29 15:53:02 
tags:
- 强化学习
- 蒙特卡洛
categories: 强化学习
mathjax: true
---

## MC简介
通过采样估计值函数。有三个优势，从真实experience中学习，从仿真环境中学习，以及每个state value的计算独立于其他state。

## Monte Carlo Methods
这一章介绍的是Monte Carlo方法，和DP不一样的是，它不需要环境的信息，只需要experience即可--从真实交互或者仿真环境中得到的state,action,reward序列都行。从真实交互中学习不需要环境的信息，从仿真环境中学习需要一个model，但是这个model只用于生成sample transition，并不需要像DP那样需要所有transition的完整概率分布。在很多情况下，生成experience sample要比显示的得到概率分布容易很多。

蒙特卡洛算法基于average sample returns估计值函数。为了保证returns是可用的，这里定义蒙特卡洛算法是episodic的，即所有的experience都有一个terminal state。只有在一个episode结束的时候，value estimate和policy才会改变。蒙塔卡洛算法可以在episode和episode实现增量式，不能在step和step之间实现增量式。(Monte Carlo methods can thus be incremental in an episode-by-episode sense, but not in a step-by-step online sense.)
蒙特卡洛算法通过从采样和average returns对每一个state-action进行评估。在一个state采取action得到的return取决于同一个episode后续状态的action，因为所有的action都是在不断学习中采取，从早期state的角度来看，这个问题是non-stationary的。为了解决non-stationary问题，采用GPI中的idea。DP从已知的MDP中计算value function，蒙特卡洛使用MDP的sample returns学习value function。然后value function和对应的policy交互获得好的value和policy。
这一章就是把DP中的各种想法推广到了MC上，比如prediction和control问题，DP使用的是整个MDP，而MC使用的是MDP的采样。


## Monte Carlo Prediction
预测问题就是估计value function，从state value function说起。最简单的想法就是使用experience估计value function，通过对每个state experience中return做个average。

### First visti MC method
这里主要介绍两个算法，一个叫做$first visit MC method$，另一个是$every visit MC method$。比如要估计策略$\pi$下的$v(s)$，通过采样一系列经过$s$的episodes，$s$在每一个episode中出现一次叫做一个$visit$，一个$s$可能在一个episode中出现多次。$first visit$就是只取第一次$visit$估计$v(s)$，$every visit$就是每一次$visit$都用。
下面给出$first visit$的算法：
**First visit MC preidction**
**输入** 被评估的policy $\pi$
**初始化**:
$\qquad V(s)\in R,\forall s \in S$
$\qquad Returns(s)=[],\forall s \in S$
**Loop** for each episeode:
$\qquad$生成一个episode
$\qquad G\leftarrow 0$
$\qquad$**Loop** for each step, $t= T-1,T-2, \cdots, 1$
$\qquad\qquad G\leftarrow G + \gamma R_t$
$\qquad\qquad$ 如果$S_t$没有在$S_0, \cdots , S_{t-1}$中出现过
$\qquad\qquad\qquad Returns(S_t).apppend(G)$
$\qquad\qquad\qquad V(S_t)\leftarrow = average(Returns(S_t))$ 
$every visit$的话，不用加上判断$S_t$是否出现过的那一句就行了。
当$s$处的$visit$趋于无穷次的时候，$first vist$和$every visit$算法都收敛于$v_{\pi}(s)$。
$first vissit$中，每一个return都是$v_{\pi}(s)$的一个有限方差独立同分布估计。通过大数定律，估计平均值（$average(Returns(S_0),\cdots, average(Returns(S_t)$）的序列收敛于它的期望。每一个average都是它自己的一个无偏估计，标准差是$\frac{1}{\sqrt{n}}$。
$every visit$的收敛更难直观的去理解，但是它二次收敛于$v\_{\pi}(s)$。
补充一点：
大数定律：无论抽象分布如何，均值服从正态分布。
中心极限定理：样本大了，抽样分布近似于整体分布。

这里再次对比一下DP和MC，在扑克牌游戏中，我们知道环境的所有信息，但是我们不知道摸到下一张牌的概率，比如我们手里有很多牌了，我们知道下一张摸到什么牌会赢，但是我们不知道这件事发生的概率。使用MC可以采样获得，所以说，即使有时候知道环境信息，MC方法可能也比DP方法好。

### MC backup diagram
能不能推广DP中的backup图到MC中？什么是backup图？backup图顶部是一个root节点，表示要被更新的节点，下面是所有的transitions，leaves是对于更新有用的reward或者estimated values。
MC中的backup图，root节点是一个state，下面是一个episode中的所有transtion轨迹，以terminal state为终止节点。
MC图和DP图的对比，DP图展示了所有可能的transitions，而MC图只展示了采样的那个episode；DP图只包含一步的transitions，而MC图包含一个episode的所有序列。
![mc backup]()
![dp backup page 59]()

### MC的特点
DP中每个state的估计都依赖于它的后继state，而MC中每个state value的计算都不依赖于任何其他state value（MC算法不进行bootstrap），所以可以单独估计某一个state或者states的一个子集。而且估计单个state的计算复杂度和states的数量无关，我们可以只取感兴趣的states子集进行评估，这是MC的第三个优势。前两个优势是从actural experience中学习和从simulated的experience中学习。

## Monte Carlo Estimation of Action Values
如果没有model的话，需要估计state-action value而不是state value。有model的话，只有state value就可以确定policy，可以估计一步，选择使reward和next_state value加起来最大的action。没有model的话，只用state value是不够的，因为不知道下一个state是什么。而只用action value是可以确定policy的，选择值最大的那个action value，采用相应的action即可。所以这一节的目标是学习action value function。
和第一节介绍的方法不同的是，第一节求解的是state value，这里换成了aciton value。有一个问题是许多state-action可能从来没有被访问过，如果$\pi$是deterministic的，每一个state输出一个action，其他action的MC估计没有returns进行平均，就无法进行更新。所以，我们需要估计每一个state对应的所有action，这是exploration问题。
对于action value的policy evaluation，必须保证continual exploration。一种实现方式是指定episode开始的state-action pair，每一个pair都有大于$0$的概率被选中,这就保证了每一个action-pair在无限个episode中会被访问无限次，这叫做exploring starts。这种假设有时候有用，但是在某些时候，我们无法控制环境产生的experience，可行的方法是使用stochastic policy。

## Monte Carlo Control
MC control使用的还是GPI的想法，估计当前policy的action value，基于action value改进policy，不断迭代。
这里考虑经典的policy iteration，执行一次完全的iterative policy evaluation，再执行一次完全的policy improvement，不断迭代。对于policy evaluation，每次evaluation都使用多个episodes的experience，每次action value都会离true value function更近。这里我们假设有无限个exploring starts生成的episodes，满足这些条件时，对于任意$\pi_k$都会精确计算出$q_{\pi_k}$。对于policy improvement，只要对于当前的action value function进行贪心即可，即：
$$\pi(s) = arg\ max_a q(s,a)$$
第$4$章给出了证明，即policy improvement theorem。在每一轮improvement中，对所有的$s\in $，执行：
\begin{align\*}
q_{\pi_k}(s,\pi_{k+1}(s)) &=q_{\pi_k}(s, argmax_a q_{\pi_k}(s,a))\\
&max_a q_{\pi_k}(s,a)\\
&\gt q_{\pi_k}(s, \pi_k(s))\\
&\gt v_{\pi_k}(s)
\end{align\*}
为了给出MC算法的收敛保证，上述算法需要满足两个假设，一个是eploring start，一个是policy evaluation需要无限个episode的experience。但是现实中，这两个条件是不可能满足的，我们需要替换掉这些条件，使得效果并不会有太大的影响。
无限个episode的假设比较容易去掉，在DP方法中也有这些问题。在DP和MC任务中，都有两种方法去掉无限episode的限制，第一种方法是像iterative policy evaluation一样，规定一个误差的bound，在每一次evaluation迭代，逼近$q_{\pi_k}$，通过足够多的迭代确保误差小于bound，可能需要很多个episode才能达到这个bound。第二种是进行不完全的policy evaluation，和DP一样，使用小粒度的policy evaluation，可以只执行iterative policy evaluation的一次迭代，也可以执行一次单个state的improvement和evaluation。对于MC方法来说，很自然的就想到基于一个episode进行evaluation和improvement。每经历一个episode，执行该episode内相应state的evaluation和improvement。
也就是说一个是规定每次迭代的bound，一个是规定每次迭代的次数。

### 伪代码
**First visit MCES**
**初始化**
$\qquad$任意初始化$\pi(s)\in A(s), \forall s\in S$
$\qquad$任意初始化$Q(s, a)\in R, \forall s\in S, \forall a \in A(s)$
$\qquad$Returns(s,a)\leftarrow$ empty list, \forall s\in S, \forall a \in A(s)$
**Loop forever(for each episode)**
$\qquad$随机选择满足$S_0\in S, A_0\in A(S_0)$的state-action$(S_0,A_0)$，满足概率大于$0$
$\qquad$从$S_0,A_0$生成策略$\pi$下的一个episode，$S_0,A_0,R_1,\cdots,S_{T-1},A_{T-1},R_T$
$\qquad G\leftarrow 0$
$\qquad$**Loop for each step of episode**,$t=T-1,T-2,\cdots,0$
$\qquad\qquad G\leftarrow \gamma G+R_{t+1}$
$\qquad\qquad$如果$S_t,A_t$没有在$S_0,A_0,\cdots, S_{t-1},A_{t-1}$中出现过
$\qquad\qquad\qquad$Returns(S_t,A_t).append(G)
$\qquad\qquad\qquad Q(S_t,A_t) \leftarrow average(Returns(S_t, A_t)$
$\qquad\qquad\qquad \pi(S_t) \leftarrow argmax_a Q(S_t,a)$
这个算法一定会收敛到全局最优解，如果收敛到一个suboptimal policy，value function会收敛到到该policy的true value function，然后再进行improvement会改进该suboptimal policy。

## Mnote Carlo Control without Exploring Starts
上节主要是去掉了无穷个episode的限制，这节需要去掉ES的限制，解决方法是需要agents一直能够去选择所有的actions。目前有两类方法实现，一种是on-policy，一种是off-policy。

### on-policy和off-policy
On-policy算法中，用于evaluation或者improvement的policy和用于决策的policy是相同的，而off-policy算法中，evaluation和improvement的policy和决策的policy是不同的。

### $\epsilon$ soft和$\epsilon$ greedy
在on-policy算法中，policy一般是soft的，整个policy整体上向一个deterministic policy偏移。
在$\epsilon$ soft算法中，只要满足$\pi(a|s)\gt 0,\forall s\in S, a\in A$即可。
在$\epsilon$ greedy算法中，用$\frac{\epsilon}{|A(s)|}$的概率选择non-greedy的action，使用$1 -\epsilon + \frac{\epsilon}{|A(s)|}$的概率选择greedy的action。
$\epsilon$ greedy是$\epsilon$ soft算法中的一类，可以看成一种特殊的$\epsilon$ soft算法。
本节介绍的on policy方法使用$\epsilon$ greedy算法。

### On policy first visit MC
本节介绍的on policy MC算法整体的思路还是GPI，首先使用first visit MC估计当前policy的action value function。去掉exploring starting条件之后，为了保证exploration，不能直接对所有的action value进行贪心，使用$\epsilon$ greedy算法保持exploration。
对于任意的$\epsilon$ soft policy$\pi$，$\epsilon，相对于$q\_{\pi}$的$\epsilon$ greedy算法至少和$\pi$一样好。
**On policy first visit MC Control**
$\epsilon \gt 0$
**初始化**
$\qquad$用任意$\epsilon$ soft算法初始化$\pi$
$\qquad$任意初始化$Q(s, a)\in R, \forall s\in S, \forall a \in A(s)$
$\qquad$Returns(s,a)\leftarrow$ empty list, \forall s\in S, \forall a \in A(s)$
**Loop forever(for each episode)**
$\qquad$根据policy $\pi$生成一个episode，$S_0,A_0,R_1,\cdots,S_{T-1},A_{T-1},R_T$
$\qquad G\leftarrow 0$
$\qquad$**Loop for each step of episode**,$t=T-1,T-2,\cdots,0$
$\qquad\qquad G\leftarrow \gamma G+R_{t+1}$
$\qquad\qquad$如果$S_t,A_t$没有在$S_0,A_0,\cdots, S_{t-1},A_{t-1}$中出现过
$\qquad\qquad\qquad$Returns(S_t,A_t).append(G)
$\qquad\qquad\qquad Q(S_t,A_t) \leftarrow average(Returns(S_t, A_t)$
$\qquad\qquad\qquad A^{\*}\leftarrow argmax_a Q(S_t,a)$
$\qquad\qquad\qquad$**For all** $a \in A(S_t):$
$\qquad\qquad\qquad\qquad\pi(a|S_t)\leftarrow \begin{cases}1-\epsilon+\frac{\epsilon}{|A(S_t)|}\qquad if a = A^{\*}\\\frac{\epsilon}{|A(S_t)|}\qquad a\neq A^{\*}\end{cases}$


## Off-policy Prediction via Importance Sampling

## Incremental Implementation

## Off-policy Monte Carlo Control

## Discounting-aware Importance Sampling

## Summary
 
