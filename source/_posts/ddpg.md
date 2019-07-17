---
title: ddpg
date: 2019-07-16 10:31:55
tags:
 - ddpg
 - policy gradient
 - 深度学习
 - 强化学习
categories: 强化学习
mathjax: true
---

## dpg
论文名称：
论文地址：

### 摘要
本文提出了deterministic policy gradient 算法。它有一个很好的形式：它是action value function的expected gradient，即deterministic policy gradient可以比stochastic policy gradient更efficiently。为了保证足够的exploration，作者引入了off-policy的actor-critic算法学习determinitic target policy。
本文的contributions，dpg比spg要好，尤其是high dimensional tasks，此外，dpg不需要消耗更多的计算资源。还有就是对于一些应用，用可导的policy，但是没有办法加noist，这种情况下dpg更合适。

### 简介
policy gradient一般用的是一个概率分布表示policy，$\pi_{\theta}(a|s) = \mathbb{P}\left[a|s; \theta]$在state $s$处，根据$\theta$参数化的policy随机选择action $a$。policy gradient算法通常通过采样这个stochastic policy，调整网络参数超reward更大的地方移动。这篇文章使用了deterministic polices $a=\mu_{\theta}(s)$，按照stochastic policy的想法，使用样本的gradient更新网络参数，作者进行了证明。

### spg vs dpg
spg中policy gradient在action和state spaces，而dpg的policy gradient仅仅在state space上进行。计算spg需要更多样本，尤其是action spaces有很多维度的情况下。
作者使用dpg推导了一个off-policy的actor-critic方法，使用可导的function approximators估计action values，然后朝估计的action value function更新policy。


## ddpg
论文名称：CONTINUOUS CONTROL WITH DEEP REINFORCEMENT LEARNING
论文地址：https://arxiv.org/pdf/1509.02971.pdf

### 摘要
本文将DQN的思路推广到continuous action domain上。DQN是离散空间，DDPG是连续空间。

### 简介
强化学习的目标是学习一个policy最大化$J=\mathbb{E}\_{r_i,s_i\sim E, a_i\sim \pi}\left[R_1\right]$的expected return。
简要回顾以下action-value的定义，它的定义是从状态s开始,采取action a，采取策略$\pi$得到的回报的期望。
$$Q^{\pi}(s_t,a_t) = \mathbb{E}\_{r_{i\ge t}, s_{i \gt t}\sim E,a_{i\gt t}\sim \pi}\left[R_t|s_t,a_t\right] \tag{1}$$
（注意，这里$R$的下标和reinforcement learning an introduction中的定义不一样，但是这个无所谓，只要在用的时候保持统一就好了。）
许多rl方法使用bellman方程递归的更新Q:
$$Q^{\pi}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma\mathbb{E}\_{a_{t+1}\sim\pi}\left[Q^{\pi}(s_{t+1},a_{t+1})\right]\right]\tag{2}$$
如果target policy是deterministic的话，用$\mu$表示，那么就可以去掉式子里面的期望，action是deterministic的而不是服从一个概率分布：
$$Q^{\mu}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma Q^{\mu}(s_{t+1},\mu(s_{t+1}))\right] \tag{3}$$
而第一个期望只和environment相关。这就意味着可以使用off-policy方法学习$Q^{\mu}$。
在DQN中，作者使用replay buffer和target network缓解了non-linear funnction approximator不稳定的问题，作者在这篇文章将它们推广到了DDPG上面。

### DDPG
直接将Q-learning推广到continuous action space是不可行的，因为action是continuous的，对其进行max等greedy操作是不可行的。这种优化方法只适合trival action spaces的情况。所以这里使用的是DPG(deterministic policy gradient)，将其推广到non-linear case，DPG是一种actor-critic的方法。
DPG使用一个参数化的actor function $\mu(s|\theta^{\mu})$作为当前的policy，它将一个states直接mapping到一个specific action。$Q(s,a)$作为critic使用Q-learning中的Bellman公式进行更新。Actor的更新直接应用chain rule到$J$的expected reutrn ，更新actor的参数如下：
\begin{align\*}
\nabla_{\theta^{\mu}} &\approx \mathbb{E}\_{s_t\sim \rho\^{\beta}}\left[\nabla_{\theta\^{\mu}}Q(s,a|\theta^Q)|\_{s=s_t, a= \mu(s_t|\theta^{\mu})}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho\^{\beta}}\left[\frac{\partial Q(s,a|\theta\^Q)}{\partial\theta\^{\mu}}|\_{s=s_t, a= \mu(s_t|\theta^{\mu})}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho\^{\beta}}\left[\frac{\partial Q(s,a|\theta\^Q)}{\partial a}|\_{s=s_t, a= \mu(s_t)}\frac{\partial \mu(s_t|\theta\^{\mu})}{\partial\theta\^{\mu}}|\_{s=s_t}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho\^{\beta}}\left[\nabla_a Q(s,a|\theta^Q)|\_{s=s_t, a= \mu(s_t)} \nabla_{\theta_{\mu}} \mu(s|\theta_{\mu})|_{s=s_t}\right]\\\\ \tag{4}
\end{align\*}
中间的两行是我自己加的，不知道对不对，DPG论文中有证明，还没有看到，等到读完以后再说补充把。

#### Contributions
本文的几个改进：
1. 使用replay buffer，
2. 使用target network解决不稳定的问题。
3. 使用了batch-normalization。
4. exploration。off policy的一个优势就是target policy和behaviour policy可以不同。本文使用的behaviour policy $\mu'$ 添加了一个从noise process $N$中采样的noise：
$$\mu(s_t) = \mu(s_t|\theta_t^{\mu}) + N \tag{5}$$

#### 算法
算法1 DDPG 
随机初始化critic 网络$Q(s,a |\theta\^Q)$，和actor网络$\mu(s|\theta\^{\mu})$的权重$\theta\^Q$和$\theta\^{\mu}$
初始化target networks　$Q'$和$\mu'$的权重$\theta^{Q'}\leftarrow \theta\^Q,\theta^{\mu'} \leftarrow \theta\^{\mu}$
初始化replay buffer $R$
**for** episode = 1, M **do**
初始化一个随机process $N$用于exploration
receive initial observation state $s_1$
for $t=1, T$ do
根据behaviour policy选择action $a_t = \mu(s_t| \theta^{\mu}) + N_t$
执行action $a_t$，得到$r_t$和$s_{t+1}$
将transition $s_t, a_t, r_t, s_{t+1}$存到$R$
从$R$中采样$N$个transition $s_i, a_i, r_i, s_{i+1}$
设置target value $y_i = r_i + \gamma Q'(s_{i+1}, \mu'(s_{i+1}|\theta\^{\mu'})|\theta\^{Q'}$
使用$L = \frac{1}{N}\sum_i(y_i-Q(s_i,a_i|\theta\^Q))^2$更新critic
使用sampled policy gradient 更新acotr:
$$\nabla_{\theta^{\mu}}\approx \frac{1}{N}\sum_i\nabla_a Q(s,a|\theta^Q)|\_{s=s_i, a=\mu(s_i)}\nabla\_{\theta\^{\mu}}\mu(s|\theta\^{\mu})|\_{s_i}$$
更新target networks:
$$\theta'\leftarrow \tau \theta + (1-\tau) \theta'$$
end for
end for


### 实验
所有任务中，都使用了low-dimensional state和high-dimensional renderings。在DQN中，为了让问题在high dimensional environment中fully observable，使用了action repeats。在agent的每一个timestep中，进行$3$个timesteps的仿真，包含repeating action以及rendering。因此agent的observation包含$9$个feature maps（RGB，每一个有3个renderings），可以让agent推理不同frames之间的differences。frames进行下采样，得到$64\times 64$的像素矩阵，然后$8$位的RGB值转化为$[0,1]$之间的float points。
在训练的时候，周期性的进行test，test时候的不需要exploration noise$。实验表明，去掉不同的组件，即contribution中的几点之后，结果都会比原来差。没有使用target network的话，结果尤其差。
作者使用了两个baselines normalized scores，第一个是naive policy，在action space中均匀的采样action得到的mean return，第二个是iLQG。normalized之后，naive policy的mean score是0，iLQG的mean score是$1$。DDPG能够学习到好的policy，在某些任务上甚至比iLQG还要好。


## 参考文献
1.https://arxiv.org/pdf/1509.02971.pdf
