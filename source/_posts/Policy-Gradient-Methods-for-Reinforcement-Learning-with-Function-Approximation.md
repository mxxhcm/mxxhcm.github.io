---
title: Policy Gradient Methods for Reinforcement Learning with Function Approximation
date: 2019-04-17 10:42:55
tags:
 - Policy Gradient
 - 强化学习
 - 论文
categories: 强化学习
mathjax: true
---

## Abstract
强化学习有三种常用的方法，第一种是基于值函数的，第二种是policy gradient，第三种是derivative-free的方法，即不利用导数的方法。基于值函数的方法在理论上证明是很难的。这篇论文提出了policy gradient的方法，直接用函数去表示策略，根据expected reward对策略参数的梯度进行更新，REINFORCE和actor-critic都是policy gradient的方法。
本文的贡献主要有两个，第一个是给出估计的action-value function或者advantage函数，梯度可以表示成experience的估计，第二个是证明了任意可导的函数表示的policy通过policy iteration都可以收敛到locl optimal policy。

## Introduction
### 值函数方法的缺点
基于值函数的方法，在估计出值函数之后，每次通过greedy算法选择action。这种方法有两个缺点。
- 基于值函数的方法会找到一个deterministic的策略，但是很多时候optimal policy可能是stochastic的。
- 某个action的估计值函数稍微改变一点就可能导致这个动作被选中或者不被选中，这种不连续是保证值函数收敛的一大障碍。

### 本文的工作
#### 用函数表示stochastic策略
本文提出的policy gradient通过使用函数直接表示stochastic策略。举例来说，用神经网络表示的一个策略，输入是state，输出是每个action选择的概率，神经网络的参数是policy的参数。用$\mathbf{\theta}$表示policy参数，用$J$表示该策略的performance measure。然后参数$\mathbf{\theta}$的更新正比于以下梯度：
$$\nabla\mathbf{\theta} \approx \alpha \frac{\partial J}{\partial \mathbf{\theta}} \tag{1}$$
这里$\alpha$是步长，按照(1)式子进行更新，$\theta$可以确保收敛到J的局部最优值对应的local optimal policy。这里$\mathbf{\theta}$的微小改变只能造成policy和state分布的微小改变。

#### 使用值函数辅助学习policy
本文证明了通过使用满足特定属性的近似值函数，使用experience可以得到(1)的一个无偏估计。另一个方法REINFORCE也找到了(1)的一个无偏估计，但是没有使用辅助的值函数。REINFORCE的学习速度要比使用值函数的方法慢很多。此外学习一个值函数，并用它取减少方差对快速学习是很重要的。

#### 证明policy iteration收敛性
本文还提出了一种方法证明基于actor-critic和policy-iteration架构方法的收敛性。在这篇文章中，他们只证明了使用通用函数逼近的policy iteration可以收敛到local optimal policy。

## Policy Gradient Therorem
智能体的在每一步的决策通过一个策略表示出来：$\pi(s,a,\mathbf{\theta})=Pr\{a_t=a|s_t=s,\mathbf{\theta}\},\forall s\in S, \forall a\in A,\mathbf{\theta}\in R^l$。假设$\pi$是可导的，即，$\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}$存在。为了方便，通常把$\pi(s,a,\mathbf{\theta})$简写为$\pi(s,a)$。
这里有两种方式定义智能体的目标，一种是平均奖励，一种是从指定状态获得的长期奖励。

### Average Reward(平均奖励)
平均奖励是，策略根据每一步的长期期望奖励$\rho(\pi)$进行排名
$$\rho(\pi) = lim_{n\rightarrow \infty}\frac{1}{n}\mathbb{E}\{r_1+r_2+\cdots+r_n|\pi\} = \sum_s d^{\pi}(s) \sum_a\pi(s,a)R_s^a.$$
其中$d^{\pi}(s) = lim_{t\rightarrow \infty} Pr\{s_t=s|s_0,\pi\}$是我们假设的策略$\pi$下的固定分布，对于所有的策略都是独立于$s_0$的。这里，我想了一天都没有想明白，为什么？？？第一个等号，我可以理解，这里$r_n$表示的是在时间步$n$的immediate reward，所以第一个等号表示的是在策略$\pi$下$n$个时间步的imediate reward平均值的期望。而第二个等号$d^{\pi}(s)$到底是什么意思，是从初始状态$s_0$到$t$时刻状态$s$的概率吗，好像就是这样子的，但是为什么可以这么算呢？第一个求和号对$s$求和，相当于算的是所有从$s_0$到$t\rightarrow\infty$的$s$的所有取值，然后再对每一个$s$，计算所有可能采取的action。
state-action value定义为：
$$Q^{\pi}(s,a) = \sum_{t=1}^{\infty}\mathbb{E}\{r_t - \rho(\pi)|s_0=s,a_0=a,\pi\}, \forall s\in S, a\in A.$$

### Long-tern Accumated Reward from Designated State(从指定状态开始的累计奖励)
这种情况是指定一个开始状态$s_0$，然后我们只关心从这个状态得到的长期reward。
$$\rho(\pi) = \mathbb{E}\{\sum_{t=1}^{\infty}\gamma^{t-1}|s_0,\pi\},$$
$$Q^{\pi}(s,a) = \mathbb{E}\{\sum_{k=1}^{\infty}r_{t+k}|s_t=s,a_t=a,\pi\}.$$
其中$\gamma\in[0,1]$是折扣因子，只有在episodic任务中才允许取$\gamma=1$。这里，我们定义$d^{\pi}(s)$是从开始状态$s_0$执行策略$\pi$遇到的状态的折扣权重：
$d^{\pi}(s) = \sum_{t=1}^{\infty}\gamma^tPr\{s_t = s|s_0,\pi\}.$
### Policy Gradient Theorem
对于任何MDP，不论是平均奖励还是指定初始状态的形式，都有：
$$\frac{\partial J}{\partial \mathbf{\theta}} = \sum_ad^{\pi}(s)\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi}(s,a), \tag{2}$$
证明：
平均奖励
\begin{align\*}
\nabla v_{\pi}(s) &= \nabla \left[ \sum_a \pi(a|s)q_{\pi}(s,a)\right], \forall s\in S \\
&= \sum_a \left[\nabla\pi(a|s)q_{\pi}(s,a)\right], \\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla q_{\pi}(s,a)\right] \\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla \left[r-\rho(\pi)+\sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right]\right] \\
&= \sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\left[-\nabla \rho(\pi)+\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right]\right], \nabla r = 0\\
\end{align\*}
而由$\sum_s\pi(s,a)=1$，我们得到：
$$\nabla v_{\pi}(s)=\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \nabla v_{\pi}(s)$$
同时在上式两边对$d^{\pi}$进行求和，得到：
$$\sum_sd^{\pi}(s)\nabla v_{\pi}(s)=\sum_sd^{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd^{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi}(s)\nabla v_{\pi}(s)$$
因为$d^{\pi}$是稳定的，
$$\sum_sd^{\pi}(s)\nabla v_{\pi}(s)=\sum_sd^{\pi}(s)\sum_a\left[\nabla\pi(a|s)q_{\pi}(s,a) + \sum_sd^{\pi}(s)\nabla \sum_{s',r}p(s',r|s,a)v_{\pi}(s')\right] - \sum_sd^{\pi}(s)\nabla v_{\pi}(s)$$
那么:
\begin{align\*}
\end{align\*}
指定初始状态$s_0$:
\begin{align\*}
\nabla v_{\pi}(s) &= \nabla \[ \sum_a \pi(a|s)q_{\pi}(s,a)\], \forall s\in S \\
&= \sum_a \[\nabla\pi(a|s)q_{\pi}(s,a)\], \forall s\in S \\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla q_{\pi}(s,a)\] \\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)(r+\gamma v_{\pi}(s'))\] \\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + \pi(a|s) \nabla \sum_{s',r}p(s',r|s,a)r + \pi(a|s)\nabla \sum_{s',r}p(s',r|s,a)\gamma v_{\pi}(s'))\] \\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\nabla v_{\pi}(s') \] \\
&= \sum_a\[\nabla\pi(a|s)q_{\pi}(s,a) + 0 + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\\
&\ \ \ \ \ \ \ \ \sum_{a'}[\nabla\pi(a'|s')q_{\pi}(s',a') + \pi(a'|s')\sum_{s''}\gamma p(s''|s',a')\nabla v_{\pi}(s''))] \],  展开\\
&= \sum_{x\in S}\sum_{k=0}^{\infty}Pr(s\rightarrow x, k,\pi)\sum_a\nabla\pi(a|x)q_{\pi}(x,a) 
\end{align\*}
第(5)式使用了$v_{\pi}(s) = \sum_a\pi(a|s)q(s,a)$进行展开。第(6)式将梯度符号放进求和里面。第(7)步使用product rule对q(s,a)求导。第(8)步利用$q_{\pi}(s, a) =\sum_{s',r}p(s',r|s,a)(r+v_{\pi}(s')$ 对$q_{\pi}(s,a)$进行展开。第(9)步将(8)式进行分解。第(10)步对式(9)进行计算，因为$\sum_{s',r}p(s',r|s,a)r$是一个定制，求偏导之后为$0$。第(11)步对生成的$v_{\pi}(s')$重复(5)-(10)步骤，得到式子(11)。如果对式子(11)中的$v_{\pi}(s)$一直展开，就得到了式子(12)。式子(12)中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，这里我有一个问题，就是为什么，$k$可以取到$\infty$，后来想了想，因为对第(11)步进行展开以后，可能会有重复的state，重复的意思就是从状态$s$开始，可能会多次到达某一个状态$x$，$k$就能取很多次，大不了$k=\infty$的概率为$0$就是了。

所以，对于$v_{\pi}(s_0)$，就有：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla_{v_{\pi}}(s_0)\\
&= \sum_{s\in S}\( \sum_{k=0}^{\infty}Pr(s_0\rightarrow s,k,\pi) \) \sum_a\nabla_{\pi}(a|s)q_{\pi}(s,a)\\
&=\sum_{s\in S}\eta(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\
&=\sum_{s'\in S}\eta(s')\sum_s\frac{\eta(s)}{\sum_{s'}\eta(s')}\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\
&=\sum_{s'\in S}\eta(s')\sum_s\mu(s)\sum_a \nabla_{\pi}(a|s)q_{\pi}(s,a)\\
&\propto \sum_{s\in S}\mu(s)\sum_a\nabla\pi(a|s)q_{\pi}(s,a)
\end{align\*}

从式子(2)可以看出来，这个梯度和$\frac{\partial d^{\pi}(s)}{\partial\mathbf{\theta}}$无关：即策略改变对于状态分布没有影响，这对于使用采样来估计梯度是很方便的。这里有点不明白，举个例子来说，如果$s$是从服从$\pi$的分布中采样的，那么$\sum_a\frac{\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi}(s,a)$就是$\frac{\partial{\rho}}{\partial\mathbf{\theta}}$的一个无边估计。通常$Q^{\pi}(s,a)$也是不知道的，需要去估计。一种方法是使用真实的returns，即$R_t = \sum_{k=1}^{\infty}r_{t+k}-\rho(\pi)$或者$R_t = \sum_{k=1}^{\infty}\gamma^{k-1}r_{t+k}-\rho(\pi)$（在指定初始状态条件下）。这就是REINFROCE方法，$\nabla\mathbf{\theta}\propto\frac{\partial\pi(s_t,a_t)}{\partial\mathbf{\theta}}R_t\frac{1}{\pi(s_t,a_t)}$（$\frac{1}{\pi(s_t,a_t)}j纠正了被$\pi$偏爱的action的oversampling）。

## Policy Gradient with Approximation(使用近似的策略梯度)
如果$Q^{\pi}$也用一个学习的函数来近似，然后我们希望用近似的函数代替式子(2)中的$Q^{\pi}$，并大致给出梯度的方向。
用$f_w:S\times A \rightarrow R$表示$Q^{\pi}$的估计值。在策略$\pi$下，更新$w$的值:$\nabla w_t\propto \frac{\partial}{\partial w}\left[\hat{Q}^{\pi}(s_t,a_t) - f_w(s_t,a_t)\right]^2 \propto \left[\hat{Q}^{\pi}(s_t,a_t) - f_w(s_t,a_t)\right]\frac{\partialf_w(s_t,a_t)}{\partial w}$，其中$\hat{Q}^{\pi}(s_t,a_t)$是$Q^{\pi}(s_t,a_t)$的一个无偏估计，可能是$R_t$，当这样一个过程收敛到local optimum，那么：
$$\sum_sd^{\pi}(s)\sum_a\pi(s,a)\left[Q^{\pi}(s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w}  = 0\tag{3}$$

### Policy Gradient with Approximation Theorem
如果$f_w$满足式子(3)，并且在某种意义上与policy parameterization兼容：
$$\frac{\partial f_w(s,a)}{\partial w} = \frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}\tag{4}$$
那么有：
$$\frac{}{} = \sum_sd^{\pi}(s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}f_w(s,a)\tag{5}$$

证明：
将(4)代入(3)得到：
\begin{align\*}
&\sum_sd^{\pi}(s)\sum_a\pi(s,a)\left[Q^{\pi}(s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w} = 0\\
&\sum_sd^{\pi}(s)\sum_a\pi(s,a)\left[Q^{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}= 0\\
&\sum_sd^{\pi}(s)\sum_a\left[Q^{\pi}(s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}= 0 \tag{6}\\
\end{align\*}
从这个式子中，我们能够从式子(6)中得到

## Convergence of Policy Iteration with Function Approximation(使用函数近似的策略迭代的收敛性)
### Policy Iteration with Function Apprpximation Theorem
用$\pi$和$f_w$表示策略和值函数的任意可导函数，并且满足式子(4)中的条件，Z
