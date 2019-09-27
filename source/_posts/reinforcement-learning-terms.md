---
title: reinforcment learning terms
date: 2019-05-29 17:59:28
tags:
 - online
 - offline
 - on-policy
 - off-policy
 - bootstrap
 - model-free
 - model-based
 - value based
 - policy gradient
 - actor critic
 - 强化学习
 - state space
 - action space
 - reward
 - return
 - advantage 
 - state value
 - action value
categories: 强化学习
mathjax: true
---
## 术语定义
更多介绍可以点击查看[reinforcement learning an introduction 第三章](https://mxxhcm.github.io/2018/12/21/reinforcement-learning-an-introduction-%E7%AC%AC3%E7%AB%A0%E7%AC%94%E8%AE%B0/)
### 状态集合
$\mathcal{S}$是有限states set，包含所有state的可能取值
### 动作集合
$\mathcal{A}$是有限actions set，包含所有action的可能取值
### 转换概率矩阵或者状态转换函数
$P:\mathcal{S}\times \mathcal{A}\times \mathcal{S} \rightarrow \mathbb{R}$是transition probability distribution，或者写成$p(s_{t+1}|s_t,a_t)$
### 奖励函数
$R:\mathcal{S}\times \mathcal{A}\rightarrow \mathbb{R}$是reward function
### 折扣因子
$\gamma \in (0, 1)$
### 初始状态分布
$\rho_0$是初始状态$s_0$服从的distribution，$s_0\sim \rho_0$
### 带折扣因子的MDP
定义为tuple $\left(\mathcal{S},\mathcal{A},P,R,\rho_0, \gamma\right)$
### 随机策略
选择action，stochastic policy表示为：$\pi_\theta: \mathcal{S}\rightarrow P(\mathcal{A})$，其中$P(\mathcal{A})$是选择$\mathcal{A}$中每个action的概率，$\theta$表示policy的参数，$\pi_\theta(a_t|s_t)$是在$s_t$处取action $a_t$的概率
### Accumulated Reward
#### 期望折扣回报
定义
$$G_t =\mathbb{E} \left[\sum_{k=t}^{\infty} \gamma^{k-t} R_{k+1}\right] \tag{1}$$
为expected discounted returns，表示从$t$时刻开始的expected discounted return；
#### 状态值函数
state value function的定义是从$t$时刻的$s_t$开始的累计期望折扣奖励：
$$V^{\pi} (s_t) = \mathbb{E}\_{a_{t}, s_{t+1},\cdots\sim \pi}\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \right] \tag{2}$$
或者有时候也定义成从$t=0$开始的expected return：
$$V^{\pi} (s_0) = \mathbb{E}\_{\pi}\left[G_0|S_0=s_0;\pi\right]=\mathbb{E}\_{\pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}|S_0=s_0;\pi \right] \tag{3}$$

#### 动作值函数
action value function定义为从$t$时刻的$s_t, a_t$开始的累计期望折扣奖励：
$$Q^{\pi} (s_t, a_t) = \mathbb{E}\_{s_{t+1}, a_{t+1},\cdots\sim\pi}\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \right] \tag{4}$$
或者有时候也定义为从$t=0$开始的return的期望：
$$Q^{\pi} (s_0, a_0) = \mathbb{E}\_{\pi}\left[G_0|S_0=s_0,A_0=a_0;\pi\right]=\mathbb{E}\_{\pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}|S_0=s_0,A_0=a_0;\pi \right] \tag{5}$$

#### 优势函数
$$A^{\pi} (s,a) = Q^{\pi}(s,a) -V^{\pi} (s) \tag{6}$$
其中$a_t\sim \pi(a_t|s_t), s_{t+1}\sim P(s_{t+1}|s_t, a_t)$。$V^{\pi} (s)$可以看成状态$s$下所有$Q(s,a)$的期望，而$A^{\pi} (s,a)$可以看成当前的单个$Q(s,a)$是否要比$Q(s,a)$的期望要好，如果为正，说明这个$Q$比$Q$的期望要好，否则就不好。
优势函数的期望是$0$：
$$\mathbb{E}_{\pi}\left[A^{\pi}(s,a)\right] = \mathbb{E}_{\pi}\left[Q^{\pi}(s,a) - V^{\pi}(s)\right] = \mathbb{E}_{\pi}\left[Q^{\pi}(s,a)\right] -  \mathbb{E}_{\pi}\left[V^{\pi}(s)\right] = V^{\pi}(s) - V^{\pi}(s) = 0$$

#### 目标函数
Agents的目标是找到一个policy，最大化从state $s_0$开始的expected return：$J(\pi)=\mathbb{E}\_{\pi} \left[G_0|\pi\right]$，或者写成：
$$\eta(\pi)= \mathbb{E}\_{s_0, a_0, \cdots\sim \pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}\right] \tag{7}$$
表示$t=0$时policy $\pi$的expected discounted return，其中$s_0\sim\rho_0(s_0)$, $a_t\sim\pi(a_t|s_t)$, $s_{t+1}\sim P(s_{t+1}|s_t,a_t)$。

#### station distribution
用$p(s_0\rightarrow s,t,\pi)$表示从$s_0$经过$t$个timesteps到$s$的概率，则[policy $\pi$下$s$服从的概率分布为](https://zhuanlan.zhihu.com/p/60257706)：
$$\rho^{\pi} (s) = \int_S \sum_{t=0}^{\infty} \gamma^{t} \rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0 \tag{8}$$
其中$\rho_0(s_0)$是初始状态$s_0$服从的概率分布，当指定$s_0$时，可以看成$\rho_0(s_0) = 1$。$\rho^{\pi} (s)$可以理解为：
$$\rho^{\pi} (s) = P(s_0 = s) +\gamma P(s_1=s) + \gamma^2 P(s_2 = s)+\cdots \tag{9}$$
表示policy $\pi$下state $s$出现的概率。在每一个timestep $t$处，$s_t=s$都有一定概率发生的，也就是式子$9$。

### Average Reward
#### station distribution
对于average reward来说，它的station distribution和accumulated reward有一些不同，用$p(s_0\rightarrow s,t,\pi)$表示从$s_0$经过$t$个timesteps到$s$的概率，则policy $\pi$下$s$服从的概率分布为：
$$\rho^{\pi} (s) = \int_S \lim_{t\rightarrow\infty}\rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0 \tag{10}$$
其中$\rho_0(s_0)$是初始状态$s_0$服从的概率分布。
#### 目标函数
定义average reward $\eta$为在state distribution $\rho^\pi $和policy $\pi_\theta$上的期望：
\begin{align\*}
\eta(\pi) &= \int_S \rho^{\pi} (s) \int_A \pi(s,a) R^{\pi}(s,a)dads\\\\
&= \mathbb{E}\_{s\sim \rho^{\pi} , a\sim \pi}\left[R^{\pi}(s,a)\right] \tag{11}\\\\ 
\end{align\*}
其中$R(s,a) = \mathbb{E}\left[ r_{t+1}|s_t=s, a_t=a\right]$，是state action pair $(s,a)$的immediate reward的期望值。

#### 动作值函数
根据average reward，给出一种新的state-action value的定义方式：
$$Q^{\pi} (s,a) = \sum_{t=0}^{\infty} \mathbb{E}\left[R_t - \eta(\pi)|s_0=s,a_0=a,\pi\right], \forall s\in S, a\in A \tag{12}$$

#### 状态值函数
Value function定义还和原来一样，形式没有变，但是因为$Q$计算方法变了，所以$V$的值也变了：
$$V^{\pi} (s) = \mathbb{E}\_{\pi(a';s)}\left[Q^{\pi}(s,a')\right] \tag{13}$$

### Accumulated Reward和Average Reward
这两种方式，accumulated reward需要加上折扣银子，而average reward不需要。我们常见的都是accumulated reward这种方式的动作值函数以及状态值函数。



## 分类方式
### online vs offline
online方法中训练数据一直在不断增加，基本上强化学习都是online的，而监督学习是offline的。

### on-policy vs off-policy
behaviour policy是采样的policy。
target policy是要evaluation的policy。
behaviour policy和target policy是不是相同的，相同的就是on-policy，不同的就是off-policy，带有replay buffer的都是off-policy的方法。

## bootstrap
当前value的计算是否基于其他value的估计值。
常见的bootstrap算法有DP，TD-gamma
MC算法不是bootstrap算法。


## value-based vs policy gradient vs actor-critic 

### value-based
values-based方法主要有policy iteration和value iteration。policy iteration又分为policy evaluation和policy improvement。
给出一个任务，如果可以使用value-based。随机初始化一个policy，然后可以计算这个policy的value function，这就叫做policy evaluation，然后根据这个value function，可以对policy进行改进，这叫做policy improvement，可以证明policy一定会更好。policy evaluation和policy improvement交替迭代，在线性case下，收敛性是可以证明的，在non-linear情况下，就不一定了。
policy iteraion中，policy evaluation每一次都要进行收敛后才进行policy improvemetn，如果policy evalution只进行一次，然后就进行一次policy improvemetn的话，也就是policy evalution的粒度变小后，就是value iteration。

### policy gradient
value-based方法只适用于discrete action space，对于contionous action space的话，就无能为力了。这个时候就有了policy gradient，给出一个state，policy gradient给出一个policy直接计算出相应的action，然后给出一个衡量action好坏的指标，直接对policy的参数求导，最后收敛之后就求解出一个使用与contionous的policy

### actor-critic
如果policy gradient的metrics选择使用value function，一般是aciton value function的话，我们把这个value function叫做critic，然后把policy叫做actor。通过value funciton Q对policy的参数求导进行优化。
critic跟policy没有关系，而critic指导actor的训练，通过链式法则实现。critic对a求偏导，a对actor的参数求偏导。

## 参考文献
1.https://stats.stackexchange.com/questions/897/online-vs-offline-learning
