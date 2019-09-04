---
title: policy gradient
date: 2019-07-16 10:31:55
tags:
 - pg
 - dpg
 - ddpg
 - maddpg
 - policy gradient
 - 深度学习
 - 强化学习
categories: 强化学习
mathjax: true
---

## policy gradient
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
$$\nabla\mathbf{\theta} \approx \alpha \frac{\partial J}{\partial \mathbf{\theta}} \tag{1}$$
这里$\alpha$是步长，按照(1)式子进行更新，$\theta$可以确保收敛到J的局部最优值对应的local optimal policy。这里$\mathbf{\theta}$的微小改变只能造成policy和state分布的微小改变。

##### 使用值函数辅助学习policy
本文证明了通过使用满足特定属性的辅助近似值函数，使用experience可以得到(1)的一个无偏估计。另一个方法REINFORCE也找到了(1)的一个无偏估计，但是没有使用辅助的值函数。REINFORCE的学习速度要比使用值函数的方法慢很多。此外学习一个值函数，并用它取减少方差对快速学习是很重要的。

##### 证明policy iteration收敛性
本文还提出了一种方法证明基于actor-critic和policy-iteration架构方法的收敛性。在这篇文章中，他们只证明了使用通用函数逼近的policy iteration可以收敛到local optimal policy。

### Policy Gradient Therorem
智能体的在每一步的action由policy决定：$\pi(s,a,\mathbf{\theta})=Pr\[a_t=a|s_t=s,\mathbf{\theta}\],\forall s\in S, \forall a\in A,\mathbf{\theta}\in \mathbb{R}^l $。假设$\pi$是可导的，即，$\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}$存在。为了方便，通常把$\pi(s,a,\mathbf{\theta})$简写为$\pi(s,a)$。
这里有两种方式定义智能体的objective，一种是average reward，一种是从指定状态获得的长期奖励。

#### Average Reward(平均奖励)
平均奖励是，策略根据每一步的长期期望奖励$\rho(\pi)$进行排名
$$\rho(\pi) = lim_{n\rightarrow \infty}\frac{1}{n}\mathbb{E}\[r_1+r_2+\cdots+r_n|\pi\] = \sum_s d^{\pi}(s) \sum_a\pi(s,a)R_sa.$$
其中$d\{\pi}(s) = lim_{t\rightarrow \infty} Pr\[s_t=s|s_0,\pi\]$是我们假设的策略$\pi$下的固定分布，对于所有的策略都是独立于$s_0$的。这里，我想了一天都没有想明白，为什么？？？第一个等号，我可以理解，这里$r_n$表示的是在时间步$n$的immediate reward，所以第一个等号表示的是在策略$\pi$下$n$个时间步的imediate reward平均值的期望。
而第二个等号中，$d{\pi}(s)$是从初始状态$s_0$经过$t$步之后所有state $s$可能取值的概率，第一个求和号对$s$求和，就相当于一个离散积分，求的是$s$的期望；然后对$a$的求和，也相当于一个离散积分，求的是关于$a$的期望，所以第二个等式后面求的其实就是$R(s,a)$的期望。
state-action value定义为：
$$Q\{\pi}(s,a) = \sum_{t=1}^{\infty} \mathbb{E}\[r_t - \rho(\pi)|s_0=s,a_0=a,\pi\], \forall s\in S, a\in A.$$

#### Long-tern Accumated Reward from Designated State(从指定状态开始的累计奖励)
这种情况是指定一个开始状态$s_0$，然后我们只关心从这个状态得到的长期reward。
$$\rho(\pi) = \mathbb{E}\[\sum_{t=1}^\{\infty} \gamma{t-1}|s_0,\pi\],$$
$$Q\{\pi}(s,a) = \mathbb{E}\[\sum_{k=1}^{\infty} r_{t+k}|s_t=s,a_t=a,\pi\].$$
其中$\gamma\in[0,1]$是折扣因子，只有在episodic任务中才允许取$\gamma=1$。这里，我们定义$d^{\pi}(s)$是从开始状态$s_0$执行策略$\pi$遇到的状态的折扣权重之和：
$d^{\pi}(s) = \sum_{t=1}^{\infty} \gamma^t Pr\[s_t = s|s_0,\pi\].$
这里的$d^{\pi}$是从$s_0$开始，到$t=\infty$之间的任意时刻所有能到达state $s$的折扣概率之和。

#### Policy Gradient Theorem
对于任何MDP，不论是平均奖励还是指定初始状态的形式，都有：
$$\frac{\partial J}{\partial \mathbf{\theta}} = \sum_ad\{\pi}(s)\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q{\pi}(s,a), \tag{2}$$
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
$$\nabla v_{\pi}(s)=\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \nabla v_{\pi}(s)$$
同时在上式两边对$d\{\pi}$进行求和，得到：
$$\sum_sd\{\pi}(s)\nabla v_{\pi}(s)=\sum_sd{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi}(s)\nabla v_{\pi}(s)$$
因为$d\{\pi}$是稳定的，
$$\sum_sd\{\pi}(s)\nabla v_{\pi}(s)=\sum_sd{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi}(s)\nabla v_{\pi}(s)$$
那么:
\begin{align\*}
\end{align\*}
指定初始状态$s_0$:
\begin{align\*}
\nabla v_{\pi}(s) &= \nabla \[ \sum_a \pi(a|s)q_{\pi}(s,a)\], \forall s\in S \\\\
&= \sum_a \[\nabla\pi(a|s)q_{\pi}(s,a)\], \forall s\in S \\\\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla q_{\pi}(s,a)\] \\\\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)(r+\gamma v_{\pi}(s'))\] \\\\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s) \nabla \sum_{s',r}p(s',r|s,a)r + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)\gamma v_{\pi}(s'))\] \\\\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\nabla v_{\pi}(s') \] \\\\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\\\\
&\ \ \ \ \ \ \ \ \sum_{a'}[\nabla\pi(a'|s')q_{\pi}(s',a') + \pi(a'|s')\sum_{s''}\gamma p(s''|s',a')\nabla v_{\pi}(s''))] \],  展开\\\\
&= \sum_{x\in S}\sum_{k=0}\{\infty}Pr(s\rightarrow x, k,\pi)\sum_a\nabla\pi(a|x)q_{\pi}(x,a) 
\end{align\*}
第(5)式使用了$v_{\pi}(s) = \sum_a\pi(a|s)q(s,a)$进行展开。第(6)式将梯度符号放进求和里面。第(7)步使用product rule对q(s,a)求导。第(8)步利用$q_{\pi}(s, a) =\sum_{s',r}p(s',r|s,a)(r+v_{\pi}(s')$ 对$q_{\pi}(s,a)$进行展开。第(9)步将(8)式进行分解。第(10)步对式(9)进行计算，因为$\sum_{s',r}p(s',r|s,a)r$是一个定制，求偏导之后为$0$。第(11)步对生成的$v_{\pi}(s')$重复(5)-(10)步骤，得到式子(11)。如果对式子(11)中的$v_{\pi}(s)$一直展开，就得到了式子(12)。式子(12)中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，这里我有一个问题，就是为什么，$k$可以取到$\infty$，后来想了想，因为对第(11)步进行展开以后，可能会有重复的state，重复的意思就是从状态$s$开始，可能会多次到达某一个状态$x$，$k$就能取很多次，大不了$k=\infty$的概率为$0$就是了。

所以，对于$v_{\pi}(s_0)$，就有：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla_{v_{\pi}}(s_0)\\\\
&= \sum_{s\in S}\( \sum_{k=0}\{\infty}Pr(s_0\rightarrow s,k,\pi) \) \sum_a\nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s\in S}\eta(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\eta(s')\sum_s\frac{\eta(s)}{\sum_{s'}\eta(s')}\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\eta(s')\sum_s\mu(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\\\
&\propto \sum_{s\in S}\mu(s)\sum_a\nabla\pi(a|s)q_{\pi}(s,a)
\end{align\*}

从式子(2)可以看出来，这个梯度和$\frac{\partial d\{\pi}(s)}{\partial\mathbf{\theta}}$无关：即策略改变对于状态分布没有影响，这对于使用采样来估计梯度是很方便的。这里有点不明白，举个例子来说，如果$s$是从服从$\pi$的分布中采样的，那么$\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q{\pi}(s,a)$就是$\frac{\partial{\rho}}{\partial\mathbf{\theta}}$的一个无边估计。通常$Q{\pi}(s,a)$也是不知道的，需要去估计。一种方法是使用真实的returns，即$R_t = \sum_{k=1}^{\infty} r_{t+k}-\rho(\pi)$或者$R_t = \sum_{k=1}^{\infty} \gamma^{k-1} r_{t+k}-\rho(\pi)$（在指定初始状态条件下）。这就是REINFROCE方法，$\nabla\mathbf{\theta}\propto\frac{\partial\pi(s_t,a_t)}{\partial\mathbf{\theta}}R_t\frac{1}{\pi(s_t,a_t)}$,$\frac{1}{\pi(s_t,a_t)}$纠正了被$\pi$偏爱的action的oversampling）。

### Policy Gradient with Approximation(使用近似的策略梯度)
如果$Q^{\pi} $也用一个学习的函数来近似，然后我们希望用近似的函数代替式子(2)中的$Q^{\pi} $，并大致给出梯度的方向。
用$f_w:S\times A \rightarrow R$表示$Q^{\pi} $的估计值。在策略$\pi$下，更新$w$的值:$\nabla w_t\propto \frac{\partial}{\partial w}\left[\hat{Q^{\pi} }(s_t,a_t) - f_w(s_t,a_t)\right]2 \propto \left[\hat{Q^{\pi} }(s_t,a_t) - f_w(s_t,a_t)\right]\frac{\partial f_w(s_t,a_t)}{\partial w}$，其中$\hat{Q^{\pi} }(s_t,a_t)$是$Q^{\pi} (s_t,a_t)$的一个无偏估计，可能是$R_t$，当这样一个过程收敛到local optimum，那么：
$$\sum_sd\{\pi}(s)\sum_a\pi(s,a)\left[Q^{\pi} (s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w}  = 0\tag{3}$$

#### Policy Gradient with Approximation Theorem
如果$f_w$满足式子(3)，并且在某种意义上与policy parameterization兼容：
$$\frac{\partial f_w(s,a)}{\partial w} = \frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}\tag{4}$$
那么有：
$$\frac{}{} = \sum_sd\{\pi}(s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}f_w(s,a)\tag{5}$$

证明：
将(4)代入(3)得到：
\begin{align\*}
&\sum_sd\{\pi}(s)\sum_a\pi(s,a)\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w} = 0\\\\
&\sum_sd\{\pi}(s)\sum_a\pi(s,a)\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}= 0\\\\
&\sum_sd\{\pi}(s)\sum_a\left[Q{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}= 0 \tag{6}\\\\
\end{align\*}
从这个式子中，我们能够从式子(6)中得到

### Convergence of Policy Iteration with Function Approximation(使用函数近似的策略迭代的收敛性)
### Policy Iteration with Function Apprpximation Theorem
用$\pi$和$f_w$表示策略和值函数的任意可导函数，并且满足式子(4)中的条件，Z

## deterministic policy gradient
论文名称：Deterministic policy gradient algorithms
论文地址：

### 摘要
本文提出了deterministic policy gradient 算法。Deterministic实际上是action value function的expected gradient，这让deterministic policy gradient比stochastic policy gradient要更efficiently。为了保证足够的exploration，作者引入了off-policy的actor-critic算法学习determinitic target policy。
本文的contributions，dpg比spg要好，尤其是high dimensional tasks，此外，dpg不需要消耗更多的计算资源。还有就是对于一些应用，有可导的policy，但是没有办法加noist，这种情况下dpg更合适。

### stochastic policy gradient vs deterministic policy gradiet
#### (sotchastic) policy gradient
Policy gradient的basic idea是用参数化的policy distribution $\pi_\theta(a|s) = \mathbb{P}\left[a|s,\theta\right]$表示policy，这个policy $\pi$在state $s$处根据$\theta$表示的policy随机的选择action $a$。Policy gradient通常对policy 进行采样，然后调整policy的参数$\theta$朝着使cumulative reward更大的方向移动。
#### deterministic policy gradient
一般来说，用$a=\mu_\theta(s)$表示deterministic policy。很自然能想到使用和stochastic policy gradient相同的方法能不能得到应用于deterministic policy gradient，即朝着使得cumulative reward更大的方向更新policy的参数。之前的一些工作认为deterministic policy gradient是不存在的，这篇文章证明了deterministic policy gradient是存在的，并且有一种非常简单的形式，可以从action value function的梯度中获得。而且在某些情况下，deterministic policy gradient可以看成stochastic policy gradient的特殊情况。

#### spg vs dpg
spg和dpg的第一个显著区别就是积分的space是不同的。Spg中policy gradient是在action和state spaces上进行积分的，而dpg的policy gradient仅仅在state space上进行积分。因此，计算spg需要更多samples，尤其是action spaces维度很高的情况下。
使用stochastic polic目的y是充分的explore整个state和action space。而在使用deterministic policy时，为了确保能够持续的进行explore，就需要使用off-policy的算法了，behaviour policy使用stochastic policy进行采样，target policy是deterministic policy。作者使用deterministic policy gradient推导出了一个off-policy的actor-critic方法，使用可导的function approximators估计action values，然后利用这个function的梯度更新policy的参数，同时为了确保policy gradient没有bias，使用而来compatible function。

### 一些terms，return,vuale function和performance objective
更多介绍可以点击查看[reinforcement learning an introduction 第三章]()
- state space 
所有state的可能取值，$S$
- action space 
所有action的可能取值，$A$
- initial state distribution
初始state服从的分布，$p_1(s_1)$
- stationary transition dynamic distribution
稳定的状态转换函数，$p(s_{t+1}|s_t,a_t)$
- reward function
$S\times A\rightarrow \mathbb{R}$，
- policy
用来选择action，stochastic policy表示为：$\pi_\theta: S\rightarrow P(A)$，其中$P(A)$是选择$A$中每个action的概率，$\theta$是policy的参数，$\pi_\theta(a_t|s_t)$是在$s_t$处取所有可能的action $a_t$的概率。
- return
return的定义是$r_t{\gamma} = \sum_{i=t}^{\infty}\gamma^{i-t}r(s_i, a_i)$，表示从$t$时刻开始的累积折扣奖励。
- value function
value function的定义是所有的累积折扣奖励，即从$t=1$一直到最后的折扣奖励。
state value function的定义是state $S_1=s,s\sim p_1$处return的期望：$V{\pi}(s) = \mathbb{E}\left[r_1^{\gamma}|S_1=s;\pi\right]$，
action value function的定义是在state $S_1=s,s\sim p_1$采取某个action后return的期望：$V{\pi}(s) = \mathbb{E}\left[r_1^{\gamma}|S_1=s,A_1=a;\pi\right]$，
- performance objective
agents的目标就是找到一个最大化初始状态return的policy：$J(\pi)=\mathbb{E}\left[r_1{\gamma}|\pi\right]$
用$p(s\rightarrow s',t,\pi)$表示从$s$经过$t$个timesteps到$s'$的概率密度。用$\rho{\pi}(s'):=\int_S \sum_{t=1}^{\infty}\gamma^{t-1}p_1(s)p(s\rightarrow s', t,\pi)ds$表示$s'$服从的概率分布，通过对所有经过$t$个timesteps能够到达$s'$的state $s$积分得到。我们可以将performance objective表示成在$\rho^\pi$和$\pi_\theta$上的期望：
\begin{align\*}
J(\pi_{\theta}) &= \int_S \rho{\pi}(s) \int_A \pi_{\theta}(s,a) r(s,a)dads\\\\
&= \mathbb{E}\_{s\sim \rho{\pi}, a\sim \pi_{\theta}}\left[r(s,a)\right]\\\\ \tag{1}
\end{align\*}
其中$p_1(s)$是初始状态$s$服从的概率分布，$\mathbb{E}_{s\sim \rho\pi}\left[\cdot\right]$表示在状态分布$\rho(s)$上的期望。注意，这里和reinforcement learing an introduction第13章中有一些不同。这里是对所有的state的return，而introdution中求得是初始状态的value期望。

### stochastic policy gradient theorem
spg的基本想法就是调整policy的参数朝着$J$的梯度方向移动。
对$J(\pi_{\theta})$对$\theta$求导，得到：
\begin{align\*}
\nabla_{\theta} J(\pi_{\theta})&=\int_S\rho\pi(s) \int_A\nabla_\theta\pi_\theta (a|s)Q^\pi(s,a) dads \\\\
&=\mathbb{E}_{s\sim \rho\pi, a\sim \pi_\theta}\left[\nabla_\theta log\pi_\theta(a|s)Q^\pi(s,a)\right] \tag{2}
\end{align\*}
这就是policy gradient，很简单。state distribution $\rho\pi(s)$取决于policy parameters，但是policy gradient不依赖于state distribution的gradient。
这个理论有很重要的实用价值，因为它将performance gradient的计算变成了一个期望。然后可以通过sampling估计这个期望。这个方法中需要使用$Q\pi(s,a)$，估计$Q$不同方法就是不同的算法，最简单的使用sample return $r_t^\gamma$估计$Q^\pi(s_t,a_t)$，就是REINFORCE算法。

### stochastic actor-critic 算法
actor-critic是一个基于policy gradient theorem的结构。actor-critic包含两个组件。一个acotr通过stochastic gradient ascent调整stochastic policy $\pi_\theta(s)$的参数$\theta$，同时使用一个action-value function $Qw(s,a)$近似$Q\pi(s,a)$, $w$是function approximation的参数。Critic一般使用policy evaluation方法进行学习，比如使用td和mc等估计action value function $Q^w(s,a)\approx Q^\pi(s,a)$。一般来说，使用$Q^w(s,a)$代替真实的$Q^\pi(s,a)$会引入bias，但是，如果function approximator是compatible，即满足以下两个条件：
1. $Qw(s,a) = \nabla_\theta log\pi_\theta(a|s)^Tw$
2. 参数$w$最小化mse: $\epsilon2(w)=\mathbb{E}_{s\sim \rho^\pi,a\sim \pi_\theta}\left[(Q^w(s.a)-Q^\pi(s,a))^2\right]$，这样就没有bias了，即：
$$\nabla_\theta J(\pi_\theta)=\mathbb{E}_{s\sim \rho\pi, a\sim \pi_\theta}\left[\nabla_\theta log\pi_\theta(a|s)Q^w(s,a)\right] \tag{3}$$

直观上来说，条件1说的是compatible function approximators是stochastic policy梯度$\nabla_\theta log\pi_\theta(a|s)$的线性features，条件2需要满足$Qw(s,a)$是$Q^\pi(s,a)$的linear regression soulution。在实际应用中，条件2会放宽，使用如TD之类policy evaluation算法更efficiently的估计value function。事实上，如果条件1和2都满足的话，整个算法等价于没有使用critic，和REINFORCE算法很像。

### off-policy actor critic
在off-policy设置中，performance objective通常改成target policy的value function在behaviour policy的state distribution上进行平均，用$\beta(a|s)$表示behaviour policy：
\begin{align\*}
J_\beta(\pi_\theta) &= \int_S\rho\beta(s) V^\pi(s)ds\\\\
&=\int_S\int_A\rho\beta(s)\pi_\theta(a|s)Q^\pi(s,a)dads
\end{align\*}
对其求导和近似，得到：
\begin{align\*}
\nabla_\theta J_\beta(\pi_\theta) &\approx \int_S\int_A\rho\beta(s)\nabla_\theta\pi_\theta(a|s) Q^\pi(s,a)dads\tag{4} \\\\
&=\mathbb{E}\_{s\sim \rho\beta, a\sim \beta}\left[\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}\nabla_\theta log\pi_\theta(a|s) Q^\pi(s,a) \right]\tag{5}
\end{align\*}
这个近似去掉了一项：$\nabla_\theta Q\pi(s,a)$，有人说这样子可以保留局部最小值。。Off-policy actor-critic算法使用behaviour policy $\beta$生成trajectories，critic off-policy的从那些trajectories中估计state value function $V^v(s)\approx v^\pi(s)$。actor使用sgd从这些trajectories中off policy的更新policy paramters $\theta$，同时使用TD-error估计$Q^\pi(s,a)$。actor和critic都是用importance sampling ratio $\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}$。

### Gradients of deterministic policies
这一节主要介绍的是deterministic policy gradient theorem。首先给出直观上的解释，然后给定formal的证明。

### action value gradients
绝大多数的model-free rl算法都属于GPI，迭代的policy evaluation和policy improvement。在contious action spaces中，greedy policy improvement是不可行的，因为在每一步都需要计算一个全局最大值。所以，一个简单的想法是使用让policy朝着$Q$的gradient方向移动，而不是全局最大化$Q$。具体而言，美誉每一个访问到的state $s$，policy parameters $\theta{k+1}$的更新正比于$\nabla_\theta Q^{\mu^k}(s, \mu_\theta(s)$。如果每一个state给出一个不同的方向，如果使用state distribution $\rho^\mu(s)$求期望，最终的方向可能会被平均了：
$$\theta{k+1} = \theta^k + \alpha \mathbb{E}_{s\sim \rho^{\mu^k}}\left[\nabla_\theta Q^{\mu^k}(s, \mu_\theta(s))\right] \tag{6}$$
通过使用chain rule，我们可以看到policy improvement可以被分解成action-value对于action的gradient和policy相对于policy parameters的gradient：
$$\theta{k+1} = \theta^k + \alpha \mathbb{E}_{s\sim \rho^{\mu^k}}\left[\nabla_\theta\mu_\theta(s)\nabla_a Q^{\mu^k}(s,a)|_{a=\mu_\theta(s0)}\right] \tag{7}$$
按照惯例来说，$\nabla_\theta\mu_\theta(s)$是一个jacobian matrix，每一列是policy的$dth$ action对于$\theta$的gradient $\nabla_\theta\left[\mu_\theta(s)\right]_d$。然而，如果改变了policy，访问不同的states，state distribution　$\rho\mu$也会改变。最终不考虑distribution的变化的话，这个方法是保证收敛的。但是这里给给出的证明，和sgd一样，不需要计算state distributiond的gradient。

### deterministic policy gradient theorem
deterministic policy定义为：$\mu_\theta: S\rightarrow A, \theta \in \mathbb{R}n$。定义performance objective $J(\mu_\theta) =\mathbb{E}\left[r_1^\gamma|\mu\right]$，定义概率分布维$p(s\rightarrow s', t,u)$以及state distribution $\rho^\mu(s)$和stochastic case一样。将performance objective写成expectation如下：
\begin{align\*}
J(\mu_\theta) & = \int_S\rho\mu(s) r(s,\mu_\theta(s)) ds\\\\
&= \mathbb{E}_{s\sim \rho\mu}\left[r(s, \mu_\theta(s))\right] \tag{8}
\end{align\*}
给出deterministic policy gradient theorem：
假设MDP满足以下条件，即$\nabla_\theta\mu_\theta(s)$和$\nabla_a Q\mu(s,a)$存在，那么deterministic policy gradient存在，
\begin{align\*}
\nabla_\theta J(\mu_\theta) &= \int_S\rho\mu(s) \nabla_\theta\mu_\theta(s)\nabla_aQ^\mu(s,a)|_{a=\mu_\theta(s)}ds\\\\
&=\mathbb{E}_{s\sim \rho\mu}\left[\nabla_\theta\mu_\theta(s)\nabla_aQ^\mu(s,a)|_{a=\mu_\theta(s)}\right] \tag{9}
\end{align\*}

### spg的limit
dpg theorem看起来和spg theorem很不像，事实上，对于一大类stochastic polices来说，dpg事实上是spg的一个特殊情况。如果使用deterministic policy $\mu_\theta:S\rightarrow A$和variance parameter $\sigma$表示某些stochastic policy $\pi_{\mu_{\theta,\sigma}}$，比如$\sigma = 0$时，$\pi_{\mu_{\theta, 0}} \equiv \mu_\theta$，当$\sigma \rightarrow 0$时，stochastic policy gradient收敛于deterministic policy gradient。
考虑一个stochastic policy $\pi_{\mu_{\theta,\sigma}}$让$\pi_{\mu_{\theta,\sog,a}}(s,a)=v_\sigma(\mu_\theta(s),a)$，其中$\sigam$是控制方差的参数，并且$v_\sigma$满足条件B.1，以及MDP满足条件A.1和A.2，那么
$$\lim_{\sigma\rightarrow 0}\nabla_\theta J(\pi_{\mu_{\theta, \sigma}}) = \nabla_\theta J(\mu_\thtea) \tag{10} $$
其中左边的gradient是标准spg的gradient，右边是dpg的gradient。
这就说明spg的很多方法同样也是适用于dpg的。

### deterministic actor-critic 
接下来使用dpg theorem推到on-policy和off-policy的actor-critic算法。从最简单的on-policy更新，使用Sarsa critic开始，然后考虑off-policy算法，使用Q-learning critic描述核心思想。这些算法在实践中可能会有收敛问题，因为function approximator引入的biases问题，以及off-policy引入的不稳定。然后介绍使用compatiable function approximation的方法以及gradient td learning。

### on-policy deterministic actor-critic

### off-policy deterministic actor-critic

### compatible function approximation

## ddpg
论文名称：CONTINUOUS CONTROL WITH DEEP REINFORCEMENT LEARNING
论文地址：https://arxiv.org/pdf/1509.02971.pdf

### 摘要
本文将DQN的思路推广到continuous action domain上。DQN是离散空间，DDPG是连续空间。

### 简介
强化学习的目标是学习一个policy最大化$J=\mathbb{E}\_{r_i,s_i\sim E, a_i\sim \pi}\left[R_1\right]$的expected return。
简要回顾以下action-value的定义，它的定义是从状态s开始,采取action a，采取策略$\pi$得到的回报的期望。
$$Q{\pi}(s_t,a_t) = \mathbb{E}\_{r_{i\ge t}, s_{i \gt t}\sim E,a_{i\gt t}\sim \pi}\left[R_t|s_t,a_t\right] \tag{1}$$
（注意，这里$R$的下标和reinforcement learning an introduction中的定义不一样，但是这个无所谓，只要在用的时候保持统一就好了。）
许多rl方法使用bellman方程递归的更新Q:
$$Q{\pi}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma\mathbb{E}\_{a_{t+1}\sim\pi}\left[Q^{\pi}(s_{t+1},a_{t+1})\right]\right]\tag{2}$$
如果target policy是deterministic的话，用$\mu$表示，那么就可以去掉式子里面的期望，action是deterministic的而不是服从一个概率分布：
$$Q{\mu}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma Q^{\mu}(s_{t+1},\mu(s_{t+1}))\right] \tag{3}$$
而第一个期望只和environment相关。这就意味着可以使用off-policy方法学习$Q{\mu}$。
在DQN中，作者使用replay buffer和target network缓解了non-linear funnction approximator不稳定的问题，作者在这篇文章将它们推广到了DDPG上面。

### DDPG
直接将Q-learning推广到continuous action space是不可行的，因为action是continuous的，对其进行max等greedy操作是不可行的。这种优化方法只适合trival action spaces的情况。所以这里使用的是DPG(deterministic policy gradient)，将其推广到non-linear case，DPG是一种actor-critic的方法。
DPG使用一个参数化的actor function $\mu(s|\theta{\mu})$作为当前的policy，它将一个states直接mapping到一个specific action。$Q(s,a)$作为critic使用Q-learning中的Bellman公式进行更新。Actor的更新直接应用chain rule到$J$的expected reutrn ，更新actor的参数如下：
\begin{align\*}
\nabla_{\theta{\mu}} &\approx \mathbb{E}\_{s_t\sim \rho^{\beta}}\left[\nabla_{\theta^{\mu}}Q(s,a|\theta^Q)|\_{s=s_t, a= \mu(s_t|\theta^{\mu})}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\frac{\partial Q(s,a|\theta^Q)}{\partial\theta^{\mu}}|\_{s=s_t, a= \mu(s_t|\theta^{\mu})}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\frac{\partial Q(s,a|\theta^Q)}{\partial a}|\_{s=s_t, a= \mu(s_t)}\frac{\partial \mu(s_t|\theta^{\mu})}{\partial\theta^{\mu}}|\_{s=s_t}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\nabla_a Q(s,a|\theta^Q)|\_{s=s_t, a= \mu(s_t)} \nabla_{\theta_{\mu}} \mu(s|\theta_{\mu})|_{s=s_t}\right]\\\\ \tag{4}
\end{align\*}
中间的两行是我自己加的，不知道对不对，DPG论文中有证明，还没有看到，等到读完以后再说补充把。

#### Contributions
本文的几个改进：
1. 使用replay buffer，
2. 使用target network解决不稳定的问题。
3. 使用了batch-normalization。
4. exploration。off policy的一个优势就是target policy和behaviour policy可以不同。本文使用的behaviour policy $\mu'$ 添加了一个从noise process $N$中采样的noise：
$$\mu(s_t) = \mu(s_t|\theta_t{\mu}) + N \tag{5}$$

#### 算法
算法1 DDPG 
随机初始化critic 网络$Q(s,a |\thetaQ)$，和actor网络$\mu(s|\theta^{\mu})$的权重$\theta^Q$和$\theta^{\mu}$
初始化target networks　$Q'$和$\mu'$的权重$\theta{Q'}\leftarrow \theta^Q,\theta^{\mu'} \leftarrow \theta^{\mu}$
初始化replay buffer $R$
**for** episode = 1, M **do**
初始化一个随机process $N$用于exploration
receive initial observation state $s_1$
for $t=1, T$ do
根据behaviour policy选择action $a_t = \mu(s_t| \theta{\mu}) + N_t$
执行action $a_t$，得到$r_t$和$s_{t+1}$
将transition $s_t, a_t, r_t, s_{t+1}$存到$R$
从$R$中采样$N$个transition $s_i, a_i, r_i, s_{i+1}$
设置target value $y_i = r_i + \gamma Q'(s_{i+1}, \mu'(s_{i+1}|\theta{\mu'})|\theta^{Q'})$
使用$L = \frac{1}{N}\sum_i(y_i-Q(s_i,a_i|\thetaQ))^2$更新critic
使用sampled policy gradient 更新acotr:
$$\nabla_{\theta{\mu}}\approx \frac{1}{N}\sum_i\nabla_a Q(s,a|\theta^Q)|\_{s=s_i, a=\mu(s_i)}\nabla\_{\theta^{\mu}}\mu(s|\theta^{\mu})|\_{s_i}$$
更新target networks:
$$\theta'\leftarrow \tau \theta + (1-\tau) \theta'$$
end for
end for


### 实验
所有任务中，都使用了low-dimensional state和high-dimensional renderings。在DQN中，为了让问题在high dimensional environment中fully observable，使用了action repeats。在agent的每一个timestep中，进行$3$个timesteps的仿真，包含repeating action以及rendering。因此agent的observation包含$9$个feature maps（RGB，每一个有3个renderings），可以让agent推理不同frames之间的differences。frames进行下采样，得到$64\times 64$的像素矩阵，然后$8$位的RGB值转化为$[0,1]$之间的float points。
在训练的时候，周期性的进行test，test时候的不需要exploration noise。实验表明，去掉不同的组件，即contribution中的几点之后，结果都会比原来差。没有使用target network的话，结果尤其差。
作者使用了两个baselines normalized scores，第一个是naive policy，在action space中均匀的采样action得到的mean return，第二个是iLQG。normalized之后，naive policy的mean score是0，iLQG的mean score是$1$。DDPG能够学习到好的policy，在某些任务上甚至比iLQG还要好。


## 参考文献
1.https://arxiv.org/pdf/1509.02971.pdf
