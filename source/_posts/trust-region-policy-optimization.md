---
title: trust region policy optimization
date: 2019-09-08 14:24:37
tags:
 - policy gradient
 - natural policy gradient
 - trust region policy optimization
 - trpo
 - ppo
 - 强化学习
 - reinforcement learning
categories: 强化学习
mathjax: true
---

## 术语定义
更多介绍可以点击查看[reinforcement learning an introduction 第三章]()
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
#### 目标函数
Agents的目标是找到一个policy，最大化从state $s_0$开始的expected return：$J(\pi)=\mathbb{E}\_{\pi} \left[G_0|\pi\right]$，或者写成：
$$\eta(\pi)= \mathbb{E}\_{s_0, a_0, \cdots\sim \pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}\right] \tag{7}$$
表示$t=0$时policy $\pi$的expected discounted return，其中$s_0\sim\rho_0(s_0)$, $a_t\sim\pi(a_t|s_t)$, $s_{t+1}\sim P(s_{t+1}|s_t,a_t)$。
### Average Reward
#### 目标函数
用$p(s_0\rightarrow s,t,\pi)$表示从$s_0$经过$t$个timesteps到$s$的概率，则policy $\pi$下$s$服从的概率分布为：
$$\rho^{\pi} (s) = \int_S \sum_{t=0}^{\infty} \gamma^{t} \rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0 \tag{8}$$
其中$\rho_0(s_0)$是初始状态$s_0$服从的概率分布。$\rho^{\pi} (s)$可以理解为：
$$\rho^{\pi} (s) = P(s_0 = s) +\gamma P(s_1=s) + \gamma^2 P(s_2 = s)+\cdots \tag{9}$$
表示policy $\pi$下state $s$出现的概率。在每一个timestep $t$处，$s_t=s$都有一定概率发生的，也就是式子$9$。定义average reward $\eta$为在state distribution $\rho^\pi $和policy $\pi_\theta$上的期望：
\begin{align\*}
\eta(\pi) &= \int_S \rho^{\pi} (s) \int_A \pi(s,a) R^{\pi}(s,a)dads\\\\
&= \mathbb{E}\_{s\sim \rho^{\pi} , a\sim \pi}\left[R^{\pi}(s,a)\right] \tag{10}\\\\ 
\end{align\*}
其中$R^(s,a)$就是accumulated reward中的动作值函数，为了和average reward中的动作值函数进行区分，就用了$R$表示。

#### 动作值函数
根据average reward，给出一种新的state-action value的定义方式：
$$Q^{\pi} (s,a) = \sum_{t=0}^{\infty} \mathbb{E}\left[R_t - \eta(\pi)|s_0=s,a_0=a,\pi\right], \forall s\in S, a\in A \tag{13}$$

#### 状态值函数
Value function定义还和原来一样，形式没有变，但是因为$Q$计算方法变了，所以$V$的值也变了：
$$V^{\pi} (s) = \mathbb{E}\_{\pi(a';s)}\left[Q^{\pi}(s,a')\right]$$

## Policy Gradient
### Abstract
强化学习有三种常用的方法，第一种是基于值函数的，第二种是policy gradient，第三种是derivative-free的方法，即不利用导数的方法。基于值函数的方法在理论上证明是很难的。这篇论文提出了policy gradient的方法，直接用函数去表示策略，根据expected reward对策略参数的梯度进行更新，REINFORCE和actor-critic都是policy gradient的方法。
本文的贡献主要有两个，第一个是给出估计的action-value function或者advantage函数，梯度可以表示成experience的估计，第二个是证明了任意可导的函数表示的policy通过policy iteration都可以收敛到locl optimal policy。

### Introduction
#### 值函数方法的缺点
基于值函数的方法，在估计出值函数之后，每次通过greedy算法选择action。这种方法有两个缺点。
- 基于值函数的方法会找到一个deterministic的策略，但是很多时候optimal policy可能是stochastic的。
- 某个action的估计值函数稍微改变一点就可能导致这个动作被选中或者不被选中，这种不连续是保证值函数收敛的一大障碍。

#### 本文的工作
##### 用函数表示stochastic policy
本文提出的policy gradient用函数表示stochastic policy。比如用神经网络表示的一个policy，输入是state，输出是每个action选择的概率，神经网络的参数是policy的参数。用$\mathbf{\theta}$表示policy参数，用$J$表示该策略的performance measure。然后参数$\mathbf{\theta}$的更新正比于以下梯度：
$$\nabla\mathbf{\theta} \approx \alpha \frac{\partial J}{\partial \mathbf{\theta}} \tag{11}$$
其中$\alpha$是正定的step size，按照式子$11$进行更新，可以确保$\theta$收敛到$J$的局部最优值对应的local optimal policy。和value based方法不同，$\mathbf{\theta}$的微小改变只能造成policy和state分布的微小改变。

##### 使用值函数辅助学习policy
本文证明了通过使用满足特定属性的辅助近似值函数，利用之前的experience就可以得到$11$的一个无偏估计。REINFORCE方法也找到了$11$的一个无偏估计，但没有使用辅助值函数，此外它的速度要比使用值函数的方法慢很多。学习一个值函数，并用它取减少方差对快速学习是很重要的。

##### 证明policy iteration收敛性
本文还提出了一种方法证明基于actor-critic和policy-iteration架构方法的收敛性。在这篇文章中，他们只证明了使用通用函数逼近的policy iteration可以收敛到local optimal policy。

### Policy Gradient Therorem
智能体每一步的action由policy $\pi$决定：$\pi(s,a,\mathbf{\theta})=Pr\left[a_t=a|s_t=s,\mathbf{\theta}\right],\forall s\in S, \forall a\in A,\mathbf{\theta}\in \mathbb{R}^l $。假设$\pi$是可导的，即$\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}$存在。为了方便，通常把$\pi(s,a,\mathbf{\theta})$简写为$\pi(s,a)$。有两种方式定义智能体的objective，一种是average reward，一种是从指定状态开始的长期奖励。

#### Average Reward(平均奖励)
平均奖励是根据每一个step的的expected reward $\eta(\pi)$对不同的policy进行排名：
$$\eta(\pi) = lim_{t\rightarrow \infty}\frac{1}{t}\mathbb{E}\left[R_1+R_2+\cdots+R_t|\pi\right] = \int_S \rho^{\pi} (s) \int_A \pi(s,a) R(s,a)dads \tag{12}$$
其中$\rho^{\pi} (s) = lim_{t\rightarrow \infty} Pr\left[s_t=s|s_0,\pi\right]$是策略$\pi$下的station distribution。第一个等号中，$R_t$表示$t$时刻的immediate reward，所以第一个等号表示的是在策略$\pi$下$t$个时间步的imediate reward平均值的期望。
第二个等号，$\rho^{\pi}(s)$是从初始状态$s_0$经过$t$步之后state $s$出现的概率。第一个积分是对$s$积分，相当于求的是$s$的期望；然后对$a$的积分，求的是$s$处各个动作$a$出现概率的期望，所以第二个等式后面求的其实就是每一步$R(s,a)$平均值的期望。其实，把$\rho$换一种写法就容易理解了：$\rho^{\pi} (s) = \int_S \sum_{t=0}^{\infty} \gamma^{t} \rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0$。
给出一种新的state-action value的定义方式：
$$Q^{\pi} (s,a) = \sum_{t=0}^{\infty} \mathbb{E}\left[R_t - \eta(\pi)|s_0=s,a_0=a,\pi\right], \forall s\in S, a\in A \tag{13}$$
value function定义还和原来一样，但是因为$Q$计算方法变了，所以$V$也跟着变了：
$$V^{\pi} (s) = \mathbb{E}\_{\pi(a';s)}\left[Q^{\pi}(s,a')\right]$$

#### Long-term Accumated Reward from Designated State(从指定状态开始的累计奖励)
第二种情况是指定初始状态$s_0$，我们计算从这个状态开始得到的累积reward：
$$\rho(\pi) = \mathbb{E}\left[\sum_{t=1}^{\infty} \gamma^{t-1} R_t|s_0,\pi\right]\tag{14}$$
相应的state-action如下：
$$Q^{\pi} (s,a) = \mathbb{E}\left[\sum_{k=1}^{\infty} r_{t+k}|s_t=s,a_t=a,\pi\right] \tag{15}$$
其中$\gamma\in[0,1]$是折扣因子，只有在episodic任务中才允许取$\gamma=1$。我们定义$d^{\pi} (s)$是从开始状态$s_0$执行策略$\pi$遇到的状态的折扣权重之和：
$d^{\pi} (s) = \sum_{t=1}^{\infty} \gamma^t Pr\left[s_t = s|s_0,\pi\right] \tag{16}$
$\rho^{\pi} $是从$s_0$开始，到$t=\infty$之间的任意时刻所有能到达state $s$的折扣概率之和。

#### Policy Gradient Theorem
对于任何MDP，不论是平均奖励还是指定初始状态的形式，都有：
$$\frac{\partial J}{\partial \mathbf{\theta}} = \sum_a d^{\pi} (s)\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a), \tag{17}$$
证明：
平均奖励
\begin{align\*}
\nabla v_{\pi}(s) &= \nabla \left[ \sum_a \pi(a|s)q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a \left[\nabla\pi(a|s)q_{\pi}(s,a)\right], \\\\
&= \sum_a \left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla q_{\pi}(s,a)\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla \left[r-\rho(\pi)+\sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right]\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\left[-\nabla \rho(\pi)+\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right]\right], \nabla r = 0\\\\
\end{align\*}
而由$\sum_s\pi(s,a)=1$，我们得到：
$$\nabla v_{\pi}(s)=\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \nabla v_{\pi}(s) \tag{18}$$
同时在上式两边对$d^{\pi} $进行求和，得到：
$$\sum_sd^{\pi} (s)\nabla v_{\pi}(s)=\sum_sd{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi} (s)\nabla v_{\pi}(s) \tag{19}$$
因为$d^{\pi} $是稳定的，
$$\sum_sd^{\pi} (s)\nabla v_{\pi}(s)=\sum_sd{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi} (s)\nabla v_{\pi}(s) \tag{20}$$
那么:
\begin{align\*}
\end{align\*}
指定初始状态$s_0$:
\begin{align\*}
\nabla v_{\pi}(s) &= \nabla \left[ \sum_a \pi(a|s)q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a \left[\nabla\pi(a|s)q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla q_{\pi}(s,a)\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)(r+\gamma v_{\pi}(s'))\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s) \nabla \sum_{s',r}p(s',r|s,a)r + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)\gamma v_{\pi}(s'))\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\nabla v_{\pi}(s') \right] \\\\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\right]\\\\
&\ \ \ \ \ \ \ \ \sum_{a'}\left[\nabla\pi(a'|s')q_{\pi}(s',a') + \pi(a'|s')\sum_{s''}\gamma p(s''|s',a')\nabla v_{\pi}(s''))] \right],  展开\\\\
&= \sum_{x\in S}\sum_{k=0}^{\infty} Pr(s\rightarrow x, k,\pi)\sum_a\nabla\pi(a|x)q_{\pi}(x,a) 
\end{align\*}
第(5)式使用了$v_{\pi}(s) = \sum_a\pi(a|s)q(s,a)$进行展开。第(6)式将梯度符号放进求和里面。第(7)步使用product rule对q(s,a)求导。第(8)步利用$q_{\pi}(s, a) =\sum_{s',r}p(s',r|s,a)(r+v_{\pi}(s')$ 对$q_{\pi}(s,a)$进行展开。第(9)步将(8)式进行分解。第(10)步对式(9)进行计算，因为$\sum_{s',r}p(s',r|s,a)r$是一个定制，求偏导之后为$0$。第(11)步对生成的$v_{\pi}(s')$重复(5)-(10)步骤，得到式子(11)。如果对式子(11)中的$v_{\pi}(s)$一直展开，就得到了式子(12)。式子(12)中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，这里我有一个问题，就是为什么，$k$可以取到$\infty$，后来想了想，因为对第(11)步进行展开以后，可能会有重复的state，重复的意思就是从状态$s$开始，可能会多次到达某一个状态$x$，$k$就能取很多次，大不了$k=\infty$的概率为$0$就是了。

所以，对于$v_{\pi}(s_0)$，就有：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla_{v_{\pi}}(s_0)\\\\
&= \sum_{s\in S}\( \sum_{k=0}^{\infty} Pr(s_0\rightarrow s,k,\pi) \) \sum_a\nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s\in S}\eta(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\eta(s')\sum_s\frac{\eta(s)}{\sum_{s'}\eta(s')}\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\eta(s')\sum_s\mu(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&\propto \sum_{s\in S}\mu(s)\sum_a\nabla\pi(a|s)q_{\pi}(s,a)
\end{align\*}

从式子(2)可以看出来，这个梯度和$\frac{\partial d^{\pi} (s)}{\partial\mathbf{\theta}}$无关：即策略改变对于状态分布没有影响，这对于使用采样来估计梯度是很方便的。这里有点不明白，举个例子来说，如果$s$是从服从$\pi$的分布中采样的，那么$\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q{\pi}(s,a)$就是$\frac{\partial{\rho}}{\partial\mathbf{\theta}}$的一个无边估计。通常$Q{\pi}(s,a)$也是不知道的，需要去估计。一种方法是使用真实的returns，即$R_t = \sum_{k=1}^{\infty} r_{t+k}-\rho(\pi)$或者$R_t = \sum_{k=1}^{\infty} \gamma^{k-1} r_{t+k}-\rho(\pi)$（在指定初始状态条件下）。这就是REINFROCE方法，$\nabla\mathbf{\theta}\propto\frac{\partial\pi(s_t,a_t)}{\partial\mathbf{\theta}}R_t\frac{1}{\pi(s_t,a_t)}$,$\frac{1}{\pi(s_t,a_t)}$纠正了被$\pi$偏爱的action的oversampling）。

### Policy Gradient with Approximation(使用近似的策略梯度)
如果$Q^{\pi} $也用一个学习的函数来近似，然后我们希望用近似的函数代替式子(2)中的$Q^{\pi} $，并大致给出梯度的方向。
用$f_w:S\times A \rightarrow R$表示$Q^{\pi} $的估计值。在策略$\pi$下，更新$w$的值:$\nabla w_t\propto \frac{\partial}{\partial w}\left[\hat{Q^{\pi} }(s_t,a_t) - f_w(s_t,a_t)\right]2 \propto \left[\hat{Q^{\pi} }(s_t,a_t) - f_w(s_t,a_t)\right]\frac{\partial f_w(s_t,a_t)}{\partial w}$，其中$\hat{Q^{\pi} }(s_t,a_t)$是$Q^{\pi} (s_t,a_t)$的一个无偏估计，可能是$R_t$，当这样一个过程收敛到local optimum，那么：
$$\sum_sd^{\pi} (s)\sum_a\pi(s,a)\left[Q^{\pi} (s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w}  = 0\tag{20}$$

#### Policy Gradient with Approximation Theorem
如果$f_w$满足式子(3)，并且在某种意义上与policy parameterization兼容：
$$\frac{\partial f_w(s,a)}{\partial w} = \frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}\tag{21}$$
那么有：
$$\frac{}{} = \sum_sd^{\pi} (s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}f_w(s,a)\tag{22}$$

证明：
将(4)代入(3)得到：
\begin{align\*}
&\sum_sd^{\pi} (s)\sum_a\pi(s,a)\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w} = 0\\\\
&\sum_sd^{\pi} (s)\sum_a\pi(s,a)\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}= 0\\\\
&\sum_sd^{\pi} (s)\sum_a\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}= 0 \tag{6}\\\\
\end{align\*}
从这个式子中，我们能够从式子(6)中得到

### Convergence of Policy Iteration with Function Approximation(使用函数近似的策略迭代的收敛性)
### Policy Iteration with Function Apprpximation Theorem
用$\pi$和$f_w$表示策略和值函数的任意可导函数，并且满足式子(4)中的条件，Z



## A Natural Policy Gradient
论文名称：A Natural Policy Gradient 
论文地址：https://papers.nips.cc/paper/2073-a-natural-policy-gradient.pdf

### 摘要
作者基于参数空间的底层结构提出了natural gradient方法，找出下降最快方向。尽管gradient方法不能过大的改变参数，它还是能够朝着选择greedy optimal action而不是更好的action方向移动。基于兼容值函数的policy iteration，在每一个improvement step选择greedy optimal action。

### 引言
直接的policy gradient在解决大规模的MDPs时很有用，这种方法基于future reward的梯度在满足约束条件的一类polices中找一个$\pi$，但是这种方法是non covariant的，简单来说，就是左右两边的维度不一致。
这篇文章基于policy的底层参数结构定义了一个metric，提出了一个covariant gradient方法，通过将它和policy iteration联系起来，可以证明natural gradient朝着选择greedy optimal action的方向移动。通过在简单和复杂的MDP中进行测试，结果表明这种方法中没有出现严重的plateau phenomenon。

### A Natural Gradient
定义average reward $\eta(\pi)$为：
$$\eta(\pi) = \sum_{s,a}\rho^{\pi} (s) \pi(a;s) R(s, a)$$
这篇文章中定义了新的state action value和value function：
$$Q^{\pi} (s,a) = \mathbb{E}\_{\pi}\left[\right] \tag{23}$$
Average reward的精确梯度是：
$$\nabla\eta(\theta) = \sum_{s,a} \rho^{\pi} (s) \nabla \pi(a;s,\theta) Q^{\pi} (s,a) \tag{24}$$
在这使用$\eta(\theta)$代替了$\eta(\pi_{\theta})$。$\eta(\theta)$下降最快的方向定义为在$d\theta$的平方长度$\vert d\theta\vert^2 $ 等于一个常数时，使得$\eta(\theta+d\theta)$最小的$d\theta$的方向。平方长度的定义和一个正定矩阵$G(\theta)$有关，即：
$$\vert\theta\vert^2 = \sum_{ij} G_{ij} (\theta)d\theta_i d\theta_j = d\theta^T G(\theta) d\theta  \tag{12}$$
可以证明，最块的梯度下降方向是$G^{-1} \nabla \eta(\theta)$。标准的policy gradient假设$\mathbf{G}=\mathbf{I}$，所以最陡的下降方向是$\nabla\eta(\theta)$。作者的想法是选择一个其他的$\mathbf{G}$，新的metric不根据坐标轴的选择而变化，而是跟着坐标参数化的mainfold变化。根据新的metric定义natural gradient。
分布$\pi(a;s,\theta)$的fisher information是：
$$\mathbf{F}_s(\theta) = \mathbb{E}\_{\pi(a;s,\theta)} \left[ \frac{\partial \log \pi(a;s,\theta)}{\partial \theta_i} \frac{\partial \log \pi(a;s,\theta)}{\partial \theta_j}\right] \tag{13}$$
显然$\mathbf{F}_s$是正定矩阵，可以证明，FIM是概率分布参数空间上的一个invariant metric。不论两个点的坐标怎么选择，它都能计算处相同的distance，所以说它是invariant。
当然，$\mathbf{F}\_s$只用了单个的$s$，而在计算average reward时，使用的是一个分布，定义metric为：
$$\mathbf{F}(\theta) = \mathbb{E}\_{\rho^{\pi} (s)} \left[\mathbb{F}_s (\theta)\right] \tag{14}$$
每一个$s$对应的单个$\mathbf{F}_s$都和MDP的transition model没有关系，期望操作引入了每一个transition model的参数。直观上来说，$\mathbf{F}_s$测量的是在$s$上的probability manifold的距离，$\mathbf{F}(\theta)$对它们进行了平均。对应的下降最快的方向是：
$$\hat{\nabla}\eta(\theta) =\mathbf{F}(\theta)^{-1} \nabla\eta(\theta)  \tag{15}$$

### The Natural Gradient 和 Policy Iteration
使用$\omega$参数化的兼容性值函数$f^{\pi} (s,a;\omega)$近似$Q^{\pi} (s,a)$。
#### 兼容性值函数
定义：
$$\psi(s,a)^{\pi} = \nabla \log \pi(a;s, \theta)\qquad f^{\pi} (s,a;\omega) = \omega^T \psi^{\pi} (s,a) \tag{16}$$
其中$\left[\nabla \log \pi(a;s, \theta)\right]\_i = \frac{\partial \log \pi(a;s, \theta)}{\partial \theta_i}$。找到最小化均方根误差函数的$\omega$，记为$\hat{\omega}$：
$$\epsilon(\omega, \pi) = \sum_{s,a}\rho^{\pi} (s)\pi(a;s,\theta)(f^{\pi} (s,a;\omega) - Q^{\pi} (s,a))^2 \tag{17}$$
如果使用$f^{\pi} $代替$Q$计算出来的grdient还是exact的，就称$f$是兼容的。

##### 定理1
如果$\hat{\omega}$是最小化均方根误差$\epsilon(w,\pi_\theta)$，可以证明：
$$\hat{\omega} = \hat{\nabla} \eta(\theta) =\mathbf{F}(\theta)^{-1} \nabla\eta(\theta) =\mathbf{F}(\theta)^{-1} \nabla\eta(\theta) \tag{18}$$

#### Greedy Policy Improvement
在greedy policy improvement的每一步，在$s$处，选择$a\in argmax_{a'} f^{\pi}(s, a';\hat{\omega})$。这一节介绍natural gradient能够找到best action，而不仅仅是一个good action。
首先考虑指数函数：$\pi(s;a,\theta) \propto e^{\mathbf{\theta}\^T \Phi_{sa}}$，其中$\Phi_{sa} \in \mathbb{R}^m $是特征向量。为什么使用指数函数，因为它是affine geometry。简单来说，就是$\pi(a;s,\theta)$的probability manifold可以被弯曲。接下来证明policy在natrual gradient方向上改进的一大步等价于进行一步greedy policy improvement的policy。

##### 定理2
对于$\pi(s;a,\theta) \propto e^{\mathbf{\theta}\^T \Phi_{sa}} $，假设$\hat{\nabla}\eta(\theta)$是非零的，并且$\hat{\omega}$最小化均方根误差。让
$$\pi_{\infty}(a;s) = lim_{\alpha\rightarrow \infty}\pi(a;s,\theta + \alpha\hat{\nabla}\eta(\theta)) \tag{19}$$
当且仅当$a\in argmax_{a'} f^{\pi} (s,a';\hat{\omega})$时，有$\pi_{\infty}(a;s)\neq 0$。
证明：
...

可以看出来natural gradient趋向于选择最好的action，而普通的gradient方法只能选出来一个更好的action。
使用指数函数的目的只是为了展示在极端情况下－－有无限大的learning rate情况下的结果，接下来是普通的参数化策略，natural gradient可以根据$Q^{\pi} (s,a)$的局部近似估计$f^{\pi}(s,a;\hat{\omega})$，近似找到局部best action。

##### 定理3
加入$\hat{\omega}$最小化估计误差，使用$\theta' = \theta + \alpha \hat{\nabla}\eta(\theta)$更新参数，可以得到：
$$\pi(a;s,\theta') = \pi(a;s,\theta)(1+f^{\pi}(a,s,\hat{\omega})) + O(\alpha^2)\tag{20}$$
证明：
...

这个相当于是根据$f^{\pi}(s,a) $选择每个state的action。当然，并不是选择greedy action就一定会改善policy，还有许多例外。

### Metrics和Curvatures
在不同的参数空间中，[fisher information](https://mxxhcm.github.io/2019/09/16/fisher-information/)都可以收敛到[海塞矩阵](https://mxxhcm.github.io/2019/09/10/Jacobian-matrix-and-Hessian-matrix/)，因此，它是[aymptotically efficient](https://mxxhcm.github.io/2019/09/18/asymptotically-efficient-%E6%B8%90%E8%BF%9B%E6%9C%89%E6%95%88%E6%80%A7/)，即到达了cramer-rao bound。
$\mathbf{F}$是$\log \pi$对应的fisher information。Fisher information 和海塞矩阵有关系，但是都需要和$\pi$联系起来。是这里考虑$\eta(\theta)$的海塞矩阵，它和$\mathbf{F}$两个之间有一定联系，但是不一样。
事实上，定义的新的$\mathbf{F}$并不会收敛到海塞矩阵。但是因为海塞矩阵一般不是正定的，所以在非局部最小处附近，它提供的curvature信息用处不大。在局部最小处使用conjugate methods会更好。

## Trust Region Policy Optimization
作者提出了optimizing policies的一个迭代算法，理论上保证可以以non-trivial steps单调改善plicy。对经过理论验证的算法做一些近似，产生一个实用算法，叫做Trust Region Policy Optimization(TRPO)。这个算法和natural policy gradient很像，并且在大的非线性网络优化问题上有很高的效率。TRPO有两个变种，single-path方法应用在model-free环境中，vine方法，需要整个system能够能够从特定的states重启，通常在仿真环境中可用。
为什么要有TRPO？
1. policy gradient计算的是expected rewards梯度的最大方向，然后朝着这个方向更新policy的参数。因为梯度使用的是一阶导数，梯度太大时容易fail，梯度太小的话更新太慢。
2. 学习率很难选择，学习率固定，梯度大容易失败，梯度小更新太慢。
3. 如何限制policy，防止它进行太大的move。然后如何将policy的改变转换到model parameter的改变上。
4. 采样效率很低。对整个trajectory进行采样，但是仅仅用于一次policy update。在一个trajectory中的states是很像的，尤其是用pixels表示时。如果在每一个timestep都改进policy的话，会一直在某一个局部进行更新，训练会变得很不稳定。

## Minorize-Maximization MM算法
![mm](mm.jpeg)
如上图所示，通过迭代的最大化下界函数局部地逼近expected reward。更详细的来说，随机的初始化$\theta$，在当前$\theta$下，找到下界$M$最接近expected reward $\eta$的点，然后将$M$的最优点作为下一次的$\theta$。不断的迭代，直到收敛到optimal policy。这样做有一个条件，就是$M$要比$\eta$容易优化。比如$M$是二次函数：
$$ax^2 + bx+c\tag{21}$$
用向量形式表示是：
$$g\cdot(\theta- \theta_{old}) - \frac{\beta}{2} (\theta- \theta_{old})^T F(\theta - \theta_{old})\tag{22}$$
是一个convex function。
为什么MM算法会收敛到optimal policy，如果$M$是下界的话，它不会跨过红线$\eta$。假设新的$\eta$中的new policy更低，那么blue线一定会越过$\eta$，和$M$是下界冲突。

## Trust Region
有两种优化方法：line search和trust region。Gradient descent是line search方法。首先确定下降的方向，然后超这个方向移动一步。而trust region中，首先确定我们想要探索的step size，然后直到在trust region中的optimal point。用$\delta$表示初始的maximum step size，作为trust region的半径：
$$max_{s\in \mathbb{R}^n} m_k(s), \qquad s.t. \vert s\vert \le \delta\tag{23}$$
$m$是原始目标函数$f$的近似，我们的目标是找到半径$\delta$范围$m$的最优点，迭代下去直到最高点。在运行时可以根据表面的曲率延伸或者压缩$\delta$控制学习的速度。如果在optimal point，$m$是$f$的一个poor approximator，收缩trust region。如果approximatation很好，就expand trust region。如果policy改变太多的话，可以收缩trust region。

## Motivation
每一次策略$\pi$的更新，都能使得$\eta(\pi)$单调递增。要是能将它写成old poliy $\pi$和new policy $\hat{\pi}$的关系式就好啦。这里就给出这样一个关系式！恩！就是！
$$\eta(\hat{\pi}) = \eta(\pi) + \mathbb{E}\_{s_0, a_0, \cdots \sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t A^{\pi}(s_t,a_t)\right] \tag{24}$$
证明：
\begin{align\*}
\mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi} }\left[\sum_{t=0}^{\infty} \gamma^t A^{\pi} (s_t,a_t) \right] &=\mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}}\left[\sum_{t=0}^{\infty} \gamma^t (Q^{\pi} (s_t,a_t) - V^{\pi} (s_t))\right]  \\\\
&=\mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t ( R_{t+1} + \gamma V^{\pi} (s_{t+1}) -  V^{\pi} (s_t))\right]  \\\\
&=\mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t R_{t+1} + \sum_{t=0}^{\infty} \gamma^t (\gamma V^{\pi} (s_{t+1}) -  V^{\pi} (s_t))\right]  \\\\
&=\mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t R_{t+1} \right]+ \mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t (\gamma V^{\pi} (s_{t+1}) -  V^{\pi} (s_t))\right]  \\\\
&=\eta(\hat{\pi}) + \mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[ -  V^{\pi} (s_0))\right]  \\\\
&=\eta(\hat{\pi}) - \mathbb{E}\_{s_0, a_0,\cdots\sim \hat{\pi}} \left[ V^{\pi} (s_0))\right]  \\\\
&=\eta(\hat{\pi}) - \eta(\pi)\\\\
\end{align\*}
将new policy $\hat{\pi}$的期望回报表示为old policy $\pi$的期望回报加上另一项，只要保证这一项是非负的即可。其中$\mathbb{E}\_{s_0, a_0,\cdots, \sim \hat{\pi}}\left[\cdots\right]$表示actions是从$a_t\sim\hat{\pi}(\cdot|s_t)$得到的。

## 用求和代替期望
代入$s$的概率分布$\rho_{\pi}(s) = P(s_0 = s) +\gamma P(s_1=s) + \gamma^2 P(s_2 = s)+\cdots, s_0\sim \rho_0$，并将期望换成求和：
\begin{align\*}
\eta(\hat{\pi}) &= \eta(\pi) + \mathbb{E}\_{s_0, a_0, \cdots \sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t A^{\pi}(s_t,a_t)\right]\\\\
&=\eta(\pi) +\sum_{t=0}^{\infty} \sum_s P(s_t=s|\hat{\pi}) \sum_a \hat{\pi}(a|s)\gamma^t A^{\pi}(s,a)\\\\
&=\eta(\pi) +\sum_s\sum_{t=0}^{\infty} \gamma^t P(s_t=s|\hat{\pi}) \sum_a \hat{\pi}(a|s)A^{\pi}(s,a)\\\\
&=\eta(\pi) + \sum_s \rho_{\hat{\pi}}(s) \sum_a \hat{\pi}(a|s) A^{\pi} (s,a) \tag{2}\\\\
\end{align\*}
从上面的推导可以看出来，任何从$\pi$到$\hat{\pi}$的更新，只要保证每个state $s$处的expected advantage是非负的，即$\sum_a \hat{\pi}(a|s) A_{\pi}(s,a)\ge 0$，就能说明$\hat{\pi}$要比$\pi$好，在$s$处，新的policy $\hat{\pi}$:
$$\hat{\pi}(s) = arg\ max_a A^{\pi} (s,a) \tag{25}$$
直到所有$s$处的$A^{\pi} (s,a)$为非正停止。当然，在实际应用中，因为各种误差，可能会有一些state的expected advantage是负的。

## $\rho\_{\pi}(s)$近似$\rho\_{\hat{\pi}}(s)$（第一次近似）
上式中包含$\rho_{\hat{\pi}}$，依赖于$\hat{\pi}$，很难直接优化，作者就进行了一个近似：
$$L_{\pi} (\hat{\pi}) = \eta(\pi) + \sum_s\rho_{\pi}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a) \tag{26}$$
$$\eta (\hat{\pi}) = \eta(\pi) + \sum_s\rho_{\hat{\pi}}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a)\tag{27}$$
在$L_{\pi}(\hat{\pi} )$中用$\rho_{\pi}(s)$代替$\rho_{\hat{\pi}}(s)$，从而忽略因为policy改变导致的state访问频率的改变。当$\pi(a|s)$可导时，用$\pi_{\theta}$表示policy，用$\theta$表示$\pi$的参数，则$L_{\pi}(\hat{\pi})$和$\eta(\hat{\pi})$的一阶导相等；当$\hat{\pi} = \pi$时，$L_{\pi}(\hat{\pi}) = \eta(\hat{\pi})\tag{28}$
$$L_{\pi_{\theta_0}} (\pi_{\theta_0}) = \eta(\pi_{\theta_0}) \tag{29}$$
$$\nabla_{\theta} L_{\pi_{\theta_0}}(\pi_{\theta})|\_{\theta=\theta_0} =\nabla_{\theta} \eta(\pi_{\theta})|\_{\theta=\theta_0}\tag{30}$$
证明：
第一个式子不需要证明，而第二个式子，左边为$\eta(\pi) + \sum_s\rho_{\pi}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a)$，右边为$\eta(\pi) + \sum_s\rho_{\hat{\pi}}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a)$，分别求它们关于$\theta$的导数。$\pi$是已知量，$\hat{\pi}$是关于$\theta$的函数，$\rho_{\hat{\pi}}$是通过样本得到的，不是关于$\hat{\pi}$的函数，最后相当于只有$\hat{\pi}(a|s)$是关于$\hat{\pi}$的函数，所以左右两边就一样了。。（！！！有疑问，就是为什么？$\rho_{\hat{\pi}}$到底是怎么求的，怎么证明）
也就是说当$\hat{\pi} = \pi$时，$L_{\pi}(\pi)$和$\eta(\pi)$是相等的，在$\pi$对应的参数$\theta$周围的无穷小范围内，可以近似认为它们依然相等。$\pi$的参数$\theta_{\pi}$进行足够小的step更新到达新的policy $\hat{\pi}$，相应参数为$\theta_{\hat{\pi}}$，在改进$L_{\pi}$同时也改进了$\eta$，但是这个足够小的step是多少是不知道的。

## conservative policy iteration
为了求出这个step到底是多少，有人提出了conservative policy iteration算法，该算法提供了$\eta$提高的一个lower bound。用$\pi_{old}$表示current policy，用$\pi'$表示使得$L_{\pi_{old}}$取得最大值的policy，$\pi' = arg\ min_{\pi'} L_{\pi_{old}}(\pi')$，新的policy $\pi_{new}$定义为：
$$\pi_{new}(a|s) = (1-\alpha) \pi_{old}(a|s)+\alpha\pi'(a|s) \tag{31}$$
可以证明，新的policy $\pi_{new}$和老的policy $\pi_{old}$之间存在以下关系：
$$\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new}) - \frac{2\epsilon \gamma}{(1-\gamma(1-\alpha))(1-\gamma)}\alpha^2 , \epsilon = max_s \vert\mathbb{E}\_{a\sim\pi'}\left[A^{\pi} (s,a)\right]\vert \tag{32}$$
证明：
进行缩放得到：
$$\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new}) - \frac{2\epsilon \gamma}{(1-\gamma)^2 }\alpha^2 \tag{33}$$

## 通用随机策略单调增加的证明
从公式$9$我们可以看出来，改进右边就一定能改进真实的performance $\eta$。然而，这个bound只适用于通过公式$7$生成的混合policy，在实践中，这类policy很少用到，而且限制条件很多。所以我们想要的是一个适用于任何stochastic policy的lower bound，通过提升这个bound提升$\eta$。
作者使用$\pi$和$\hat{\pi}$之间的一个距离代替$\alpha$，将公式$8$扩展到了任意stochastic policy，而不仅仅是混合policy。这里使用的distance measure，叫做total variation divergence，对于离散的概率分布$p,q$来说，定义为：
$$D_{TV}(p||q) = \frac{1}{2} \sum_i \vert p_i -q_i \vert \tag{34}$$
定义$D_{TV}^{max}(\pi, \hat{\pi})$为：
$$D_{TV}^{max} (\pi, \hat{\pi}) = max_s D_{TV}(\pi(\cdot|s) || \hat{\pi}(\cdot|s))\tag{35}$$
让$\alpha = D_{TV}^{max}(\pi_{old}, \pi_{new})$，新的bound如下：
$$\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new}) - \frac{4\epsilon \gamma}{(1-\gamma)^2 }\alpha^2 , \qquad\epsilon = max_{s,a} \vert A^{\pi}(s,a)\vert \tag{36}$$
证明：
...

Total variation divergence和KL散度之间有这样一个关系：
$$D_{TV}(p||q)^2 \le D_{KL}(p||q) \tag{37}$$
证明：
...
让
$$D_{KL}^{max}(\pi, \hat{\pi}) = max_s D_{KL}(\pi(\cdot|s)||\hat{\pi}(\cdot|s)) \tag{38}$$
从公式$12$中可以直接得到：
\begin{align\*}
\eta(\hat{\pi}) &\ge L_{\pi}(\hat{\pi}) - \frac{4\epsilon \gamma}{(1-\gamma)^2 }\alpha^2 \\\\
&\ge L_{\pi}(\hat{\pi}) - \frac{4\epsilon \gamma}{(1-\gamma)^2 }D_{KL}^{max}(\pi, \hat{\pi}) \\\\
& \ge L_{\pi}(\hat{\pi}) - CD_{KL}^{max}(\pi, \hat{\pi}), C=\frac{4\epsilon \gamma}{(1-\gamma)^2} \tag{15}
\end{align\*}
根据公式$12$，我们能生成一个单调非递减的sequence：$\eta(\pi_0)\le \eta(\pi_1) \le \eta(\pi_2) \le \cdots$，记$M_i(\pi) = L_{\pi_i}(\pi) - CD_{KL}^{max}(\pi_i, \pi)$，有：
因为：
$$\eta(\pi_{i+1}) \ge M_i(\pi_{i+1})\tag{39}$$
$$\eta(\pi_i) = M_i(\pi_i)\tag{40}$$
上面的第一个式子减去第二个式子得到：
$$\eta(\pi_{i+1}) - \eta(\pi_i)\ge M_i(\pi_{i+1})-M_i(\pi_i) \tag{41}$$
在每一次迭代的时候，确保$M_i(\pi_{i+1}) - M_i(\pi_i)\ge 0$就能够保证$\eta$是非递减的，最大化$M_i$就能实现这个目标，$M_i$是miorize $\eta$的近似目标。这种算法是minorizaiton maximization的一种。

## 参数化策略的优化（第二次近似）
前面几小节考虑的optimization问题时没有考虑$\pi$的参数化，并且假设所有的states都可以被evaluated。这一节介绍如何在有限的样本下和任意的参数化策略下，从理论基础推导出一个实用的算法。
用$\theta$表示参数化策略$\pi_{\theta}(a|s)$的参数$\theta$，将目标表示成$\theta$而不是$\pi$的函数，即用$\eta(\theta)$表示原来的$\eta(\pi_\theta)$，用$L_{\theta}(\hat{\theta})$表示$L_{\pi_{\theta}}(\pi_{\hat{\theta}})$，用$D_{KL}(\theta||\hat{\theta})$表示$D_{KL}(\pi_{\theta}||\pi_{\hat{\theta}})$。用$\theta_{old}$表示我们想要改进的policy参数。
上一小节我们得到$\eta(\theta) \ge L_{\theta_{old}}(\theta) - CD_{KL}^{max}(\theta_{old}, \theta)$，当$\theta = \theta_{old}$时取等。通过最大化等式右边，可以提高$\eta$的下界：
$$maximize_{\theta}\left[L_{\theta_{old}}(\theta) - CD_{KL}^{max}(\theta_{old}, \theta)\right]\tag{42}$$
在实践中，如果使用上述理论中的penalty coefficient $C$，会导致steps size很小。一种方法是使用new policy 和old policy之间的KL散度进行约束，可以采取更大的steps，这个约束叫做trust region constraint:
$$maxmize_{\theta} L_{\theta_{old}} (\theta),\qquad s.t. D_{KL}^{max}(\theta_{old},\theta) \le \delta \tag{43}$$
这样会在state space的每一个state都有一个KL散度约束。由于约束太多，这个问题还是不能解。这里使用average KL divergence进行近似:
$$\bar{D}\_{KL}^{\rho}(\theta_1, \theta_2) = \mathbb{E}\_{s\sim \rho}\left[D_{KL}(\pi_{\theta_1}(\cdot|s) || \pi_{\theta_2}(\cdot|s))\right] \tag{44}$$
公式$15$变成：
$$maxmize_{\theta} L_{\theta_{old}} (\theta), \qquad s.t. \bar{D}\_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta \tag{45}$$

## 目标函数和约束的采样估计（第三次近似）
上一节介绍的是关于policy parameter的有约束优化问题，约束条件为每一次policy更新时限制policy变化的大小，优化expected toral reward $\eta$的一个估计值。这一节使用Monte Carlo仿真近似目标和约束函数。
代入$L_{\theta_{old}}$的等式，得到：
$$maxmize_{\theta}\sum_s \rho_{\theta_{old}}(s) \sum_a\pi_{\theta}(a|s)A_{\theta_{old}}(s,a), \qquad s.t. \bar{D}\_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta \tag{46}$$
首先用期望$\frac{1}{1-\gamma}\mathbb{E}\_{s\sim \rho_{\theta_{old}}}\left[\cdots\right]$代替目标函数中的$\sum_s\rho_{\theta_{old}}(s) \left[\cdots\right]$。接下来用$Q$值$Q_{\theta_{old}}$代替advantage $A_{\theta_{old}}$，结果多了一个常数项，不影响。最后使用importance smapling代替actions上的求和。使用$q$表示采样分布，$q$分布中单个的$s_n$对于loss函数的贡献在于：
$$\sum_a \pi_{\theta}(a|s_n) A_{\theta_{old}}(s_n,a) = \mathbb{E}\_{a\sim q}\left[\frac{\pi_{\theta} (a|s_n) }{q(a|s_n)}A_{\theta_{old}}(s_n,a) \right]\tag{47}$$
上面的公式就是使用importance sampling代替求和。将$A$展开：
\begin{align\*}
\sum_a \pi_{\theta}(a|s) A_{\theta_{old}}(s,a) &= \sum_a \pi_{\theta}(a|s)\left( Q_{\theta_{old}}(s,a)  - V_{\theta_{old}}(s)\right)\\\\
&= \sum_a \pi_{\theta}(a|s)Q_{\theta_{old}}(s,a)- \sum_a \pi_{\theta}(a|s)V_{\theta_{old}}(s)\\\\
&= \sum_a \pi_{\theta}(a|s)Q_{\theta_{old}}(s,a)- V_{\theta_{old}}(s)\\\\
\end{align\*}
将公式$17$的优化问题转化为：
$$maxmize_{\theta} \mathbb{E}\_{s\sim\rho_{\theta_{old}}, a\sim q}\left[\frac{\pi_{\theta} (a|s) }{q(a|s)}Q_{\theta_{old}}(s,a)\right] \qquad s.t. \mathbb{E}\_{s\sim \rho_{\theta_{old}}}\left[D_{KL}(\pi_{\theta_{old}}(\cdot|s)||\pi_{\theta}(\cdot|s))\right]\le \delta \tag{48}$$
接下来要做的就是用采样代替期望，用经验估计代替$Q$值。接下来会介绍两种方法进行估计。

第一个叫做single path，通常用在policy gradient estimation，基于单个轨迹的采样。第二个叫做vine，构建一个rollout set，从rollout set的每一个state处执行多个actions。这种方法经常用在policy iteration方法上。

### Single Path
采样$s_0\sim \rho_0$，模拟policy $\pi_{\theta_{old}}$一些timesteps生成一个trajectory $s_0, a_0, s_1, a_1, \cdots, s_{T-1}, a_{T-1}, s_T$，因此$q(a|s) = \pi_{\theta_{old}}(a|s)$。根据trajectory对每一个state action pair $(s_t,a_t)$计算$Q_{\theta_{old}}(s,a)$。

### Vine
采样$s_0\sim \rho_0$，模拟policy $\pi_{\theta_i}$生成一系列trajectories。在这些trajectories选择一个具有$N$个states的子集，表示为$s_1, c\dots, s_N$，这个集合称为rollout set。对于rollout set中的每一个state $s_n$，根据$a_{n,k}\sim q(\cdot|s_n)$采样$K$个actions。任何$q(\cdot|s_n)$都行，在实践中，$q(\cdot|s_n) = \pi_{\theta_i}(\cdot|s_n)$适用于contionous problems，像机器人运动；而均匀分布适用于离散任务，如Atari游戏。
对于$s_n$处的每一个action $a_{n,k}$，从$s_n$和$a_{n,k}$处进行rollout，估计$\hat{Q}\_{\theta_i}(s_n, a_{n,k})$。在小的有限action spaces情况下，我们可以对从给定状态任何可能的action生成一个rollout，单个$s_n$对$L_{\theta_{old}}$的贡献如下：
$$L_n(\theta) = \sum_{k=1}^K \pi_{\theta} (a_k|s_n) \hat{Q}(s_n, a_k)\tag{49}$$
其中action space是$\mathcal{A} = \{a_1, a_2,\cdots, a_K\}$。在大的连续state space中，可以使用importance sampling构建一个新的目标近似。从$s_n$处计算的$L_{\theta_{old}}$的self-normalized 估计是：
$$L_n(\theta) = \frac{\sum_{k=1}^K \frac{\pi_{\theta}(a_{n,k}|s_n)}{\pi_{\theta_{old}}(a_{n,k}|s_n)}\hat{Q}(s_n, a_{n,k}}{\sum_{k=1}^K \frac{\pi_{\theta}(a_{n,k}|s_n)}{\pi_{\theta_{old}}(a_{n,k}|s_n)}}\tag{50}$$
假设在$s_n$处执行了$K$个actions $a_{n,1}, a_{n,2}, \cdots, a_{n,K}$。Self-normalized 估计去掉了$Q$值baseline的需要。在$s_n\sim \rho(\pi)$上做平均，可以得到$L_{\theta_{old}}$和它的gradient的估计。
Vine比single path好的地方在于，给定相同数量的$Q$样本，目标函数的局部估计有更低的方差，也就是vine能更好的估计advantage。Vine的缺点在于，需要执行更多steps的模拟计算相应的advantage。此外，vine方法需要对rollout set 中的每一个state都生成多个trajectories，这就需要整个system可以重置到任意的一个state，而single path算法不需要，可以直接应用在真实的system中。

## 实用算法
使用上面介绍的single path或者vine进行采样，给出两个算法。重复执行以下步骤：
1. 使用single path或者vine算法产生一系列state-action pairs，使用Monte Carlo估计相应的$Q$值；
2. 利用样本计算公式$18$中目标函数和约束函数的估计值
3. 求出有约束优化问题的近似解，更新policy参数$\theta$，使用共轭梯度和line search。

对于$3$来说，计算KL散度的Hessian矩阵而不是协方差矩阵的梯度，生成Fisher information matrix。即使用$\frac{1}{N}\sum_{n=1}^N \frac{\partial^2}{\partial \theta_j}D_{KL}(\pi_{\theta_{old}}(\cdot|s_n)||\pi_{\theta}(\cdot|s_n))$近似$A_{ij}$而不是$\frac{1}{N}\sum_{n=1}^N \frac{\partial}{\partial \theta_i}log(\pi_{\theta}(a_n|s_n))\frac{\partial}{\partial \partial_j}log(\pi_{\theta}(a_n|s_n))$。
在前面介绍的理论和本节介绍的算法之间有以下关联：
1. 理论上验证了优化带有KL散度penalty的目标函数可以保证policy improvement是单调递增的。这个很大的penalty系数$C$会产生很小的step，所以我们想要减小这个系数。经验上来讲，很难选择一个鲁邦的penalty系数，所以我们使用一个KL散度上的一个hard constraint而不是一个penalty。
2. $D_{KL}^{max}(\theta_{old}, \theta)$是很难计算和估计的，所以将约束条件改成对期望$\bar{D}\_{KL}(\theta_{old}, \theta)$进行约束。
3. 本文的理论忽略了advantage function的近似误差。

## 和前面工作的联系
本问的推导结果和一些之前的方法有联系，他们可以统一在policy update框架下。The natural policy gradient可以看成公式$16$的一个特例：使用$L$的一个linear approximation，和$\bar{D}\_{KL}$的一个二次估计，就变成了下面的优化问题：
$$maximize_{\theta} \left[\nabla_{\theta}L_{\theta_{old}}(\theta)|\_{\theta=\theta_{old}}\cdot (\theta-\theta_{old}) \right] \qquad s.t. \frac{1}{2}(\theta_{old}-\theta)^A(\theta_{old})(\theta_{old} - \theta)\le\delta\tag{51}$$
其中$A(\theta_{old})\_{ij} = \frac{\partial}{\partial\theta_i}\frac{\partial}{\partial \theta_j}\mathbb{E}\_{s\sim \rho_{\pi}}\left[D_{KL}(\pi(\cdot|s, \theta_{old})||\pi(\cdot|s, \theta))\right]\_{\theta=\theta_{old}}$，更新公式是$\theta_{new} = \theta_{old}+\frac{1}{\lambda}A(\theta_{old})^{-1} \nabla_{\theta}L(\theta)|\_{\theta=\theta_{old}}$，其中步长$\frac{1}{\lambda}$可以看成算法参数。这和trpo不同，在每一次更新都有constraint。尽管这个差别很小，实验表明它能改善在更大规模问题上算法的性能。
同样，也可以使用$l2$约束，推导出标准的policy gradient如下：
$$maximize_{\theta} \left[\nabla_{\theta} L_{\theta_{old}}(\theta)|\_{\theta=\theta_{old}}\cdot (\theta- \theta_{old}) \qquad s.t. \frac{1}{2}\vert \theta-\theta_{old}\vert^2 \le \delta\right]$\tag{52}$$

## 参考文献
Trust Region Policy Optimization
1.http://joschu.net/docs/thesis.pdf
2.https://arxiv.org/pdf/1502.05477.pdf
3.https://medium.com/@jonathan_hui/rl-trust-region-policy-optimization-trpo-explained-a6ee04eeeee9
4.https://medium.com/@jonathan_hui/rl-trust-region-policy-optimization-trpo-part-2-f51e3b2e373a
5.https://people.eecs.berkeley.edu/~pabbeel/cs287-fa09/readings/KakadeLangford-icml2002.pdf
6.https://drive.google.com/file/d/0BxXI_RttTZAhMVhsNk5VSXU0U3c/view
7.https://zhuanlan.zhihu.com/p/26308073
8.https://zhuanlan.zhihu.com/p/60257706
