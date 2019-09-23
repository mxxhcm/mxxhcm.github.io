---
title: deterministic policy gradient
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

### stochastic policy gradient theorem
spg的基本想法就是调整policy的参数朝着$J$的梯度方向移动。
对$J(\pi_{\theta})$对$\theta$求导，得到：
\begin{align\*}
\nabla_{\theta} J(\pi_{\theta})&=\int_S\rho\pi(s) \int_A\nabla_\theta\pi_\theta (a|s)Q^{\pi} (s,a) dads \\\\
&=\mathbb{E}\_{s\sim \rho\pi, a\sim \pi_\theta}\left[\nabla_\theta log\pi_\theta(a|s)Q^{\pi} (s,a)\right] \tag{2}
\end{align\*}
这就是policy gradient，很简单。state distribution $\rho\pi(s)$取决于policy parameters，但是policy gradient不依赖于state distribution的gradient。
这个理论有很重要的实用价值，因为它将performance gradient的计算变成了一个期望。然后可以通过sampling估计这个期望。这个方法中需要使用$Q\pi(s,a)$，估计$Q$不同方法就是不同的算法，最简单的使用sample return $ r_t^{\gamma} $估计$Q^{\pi} (s_t,a_t)$，就是REINFORCE算法。

### stochastic actor-critic 算法
actor-critic是一个基于policy gradient theorem的结构。actor-critic包含两个组件。一个acotr通过stochastic gradient ascent调整stochastic policy $\pi_\theta(s)$的参数$\theta$，同时使用一个action-value function $Qw(s,a)$近似$Q\pi(s,a)$, $w$是function approximation的参数。Critic一般使用policy evaluation方法进行学习，比如使用td和mc等估计action value function $Q^w (s,a)\approx Q^\pi (s,a)$。一般来说，使用$Q^w (s,a)$代替真实的$Q^\pi (s,a)$会引入bias，但是，如果function approximator是compatible，即满足以下两个条件：
1. $Qw(s,a) = \nabla_\theta log\pi_\theta(a|s)^T w$
2. 参数$w$最小化mse: $\epsilon2(w)=\mathbb{E}\_{s\sim \rho^\pi ,a\sim \pi_\theta}\left[(Q^w (s.a)-Q^\pi (s,a))^2 \right]$，这样就没有bias了，即：
$$\nabla_\theta J(\pi_\theta)=\mathbb{E}\_{s\sim \rho\pi, a\sim \pi_\theta}\left[\nabla_\theta log\pi_\theta(a|s)Q^w (s,a)\right] \tag{3}$$

直观上来说，条件1说的是compatible function approximators是stochastic policy梯度$\nabla_\theta log\pi_\theta(a|s)$的线性features，条件2需要满足$Qw(s,a)$是$Q^\pi (s,a)$的linear regression soulution。在实际应用中，条件2会放宽，使用如TD之类policy evaluation算法更efficiently的估计value function。事实上，如果条件1和2都满足的话，整个算法等价于没有使用critic，和REINFORCE算法很像。

### off-policy actor critic
在off-policy设置中，performance objective通常改成target policy的value function在behaviour policy的state distribution上进行平均，用$\beta(a|s)$表示behaviour policy：
\begin{align\*}
J_\beta(\pi_\theta) &= \int_S\rho\beta(s) V^\pi (s)ds\\\\
&=\int_S\int_A\rho\beta(s)\pi_\theta(a|s)Q^\pi (s,a)dads
\end{align\*}
对其求导和近似，得到：
\begin{align\*}
\nabla_\theta J_\beta(\pi_\theta) &\approx \int_S\int_A\rho\beta(s)\nabla_\theta\pi_\theta(a|s) Q^\pi (s,a)dads\tag{4} \\\\
&=\mathbb{E}\_{s\sim \rho\beta, a\sim \beta}\left[\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}\nabla_\theta log\pi_\theta(a|s) Q^\pi (s,a) \right]\tag{5}
\end{align\*}
这个近似去掉了一项：$\nabla_\theta Q\pi(s,a)$，有人说这样子可以保留局部最小值。。Off-policy actor-critic算法使用behaviour policy $\beta$生成trajectories，critic off-policy的从那些trajectories中估计state value function $V^v (s)\approx v^\pi (s)$。actor使用sgd从这些trajectories中off policy的更新policy paramters $\theta$，同时使用TD-error估计$Q^\pi (s,a)$。actor和critic都是用importance sampling ratio $\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}$。

### Gradients of deterministic policies
这一节主要介绍的是deterministic policy gradient theorem。首先给出直观上的解释，然后给定formal的证明。

### action value gradients
绝大多数的model-free rl算法都属于GPI，迭代的policy evaluation和policy improvement。在contious action spaces中，greedy policy improvement是不可行的，因为在每一步都需要计算一个全局最大值。所以，一个简单的想法是使用让policy朝着$Q$的gradient方向移动，而不是全局最大化$Q$。具体而言，美誉每一个访问到的state $s$，policy parameters $\theta^{k+1} $的更新正比于$\nabla_{\theta} Q^{ {\mu}^k } (s, \mu_{\theta}(s) )$。如果每一个state给出一个不同的方向，如果使用state distribution $\rho^{\mu} (s)$求期望，最终的方向可能会被平均了：
$$\theta^{k+1} = \theta^k + \alpha \mathbb{E}\_{s\sim \rho^{ {\mu}^k } }\left[\nabla_\theta Q^{ {\mu}^k } (s, \mu_{\theta}(s))\right] \tag{6}$$
通过使用chain rule，我们可以看到policy improvement可以被分解成action-value对于action的gradient和policy相对于policy parameters的gradient：
$$\theta{k+1} = \theta^k + \alpha \mathbb{E}\_{s\sim \rho^{ {\mu}^k } }\left[\nabla_{\theta}\mu_{\theta}(s)\nabla_a Q^{ {\mu}^k } (s,a)|\_{a=\mu_\theta(s_0)}\right] \tag{7}$$
按照惯例来说，$\nabla_{\theta}\mu_{\theta}(s)$是一个jacobian matrix，每一列是policy的$dth$ action对于$\theta$的gradient $\nabla_\theta\left[\mu_\theta(s)\right]_d$。然而，如果改变了policy，访问不同的states，state distribution　$\rho\mu$也会改变。最终不考虑distribution的变化的话，这个方法是保证收敛的。但是这里给给出的证明，和sgd一样，不需要计算state distributiond的gradient。

### deterministic policy gradient theorem
deterministic policy定义为：$\mu_\theta: S\rightarrow A, \theta \in \mathbb{R}n$。定义performance objective $J(\mu_\theta) =\mathbb{E}\left[r_1^\gamma |\mu\right]$，定义概率分布维$p(s\rightarrow s', t,u)$以及state distribution $\rho^\mu (s)$和stochastic case一样。将performance objective写成expectation如下：
\begin{align\*}
J(\mu_\theta) & = \int_S\rho\mu(s) r(s,\mu_\theta(s)) ds\\\\
&= \mathbb{E}\_{s\sim \rho\mu}\left[r(s, \mu_\theta(s))\right] \tag{8}
\end{align\*}
给出deterministic policy gradient theorem：
假设MDP满足以下条件，即$\nabla_\theta\mu_\theta(s)$和$\nabla_a Q\mu(s,a)$存在，那么deterministic policy gradient存在，
\begin{align\*}
\nabla_\theta J(\mu_\theta) &= \int_S\rho\mu(s) \nabla_\theta\mu_\theta(s)\nabla_aQ^\mu (s,a)|\_{a=\mu_\theta(s)}ds\\\\
&=\mathbb{E}\_{s\sim \rho\mu}\left[\nabla_\theta\mu_\theta(s)\nabla_aQ^\mu (s,a)|\_{a=\mu_\theta(s)}\right] \tag{9}
\end{align\*}

### spg的limit
dpg theorem看起来和spg theorem很不像，事实上，对于一大类stochastic polices来说，dpg事实上是spg的一个特殊情况。如果使用deterministic policy $\mu_\theta:S\rightarrow A$和variance parameter $\sigma$表示某些stochastic policy $\pi_{\mu_{\theta,\sigma}}$，比如$\sigma = 0$时，$\pi_{\mu_{\theta, 0}} \equiv \mu_\theta$，当$\sigma \rightarrow 0$时，stochastic policy gradient收敛于deterministic policy gradient。
考虑一个stochastic policy $\pi_{\mu_{\theta,\sigma}}$让$\pi_{\mu_{\theta,\sigma}}(s,a)=v_\sigma(\mu_\theta(s),a)$，其中$\sigma$是控制方差的参数，并且$v_\sigma$满足条件B.1，以及MDP满足条件A.1和A.2，那么
$$\lim_{\sigma\rightarrow 0}\nabla_\theta J(\pi_{\mu_{\theta, \sigma}}) = \nabla_\theta J(\mu_\theta) \tag{10} $$
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
$$Q{\pi}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma\mathbb{E}\_{a_{t+1}\sim\pi}\left[Q^{\pi} (s_{t+1},a_{t+1})\right]\right]\tag{2}$$
如果target policy是deterministic的话，用$\mu$表示，那么就可以去掉式子里面的期望，action是deterministic的而不是服从一个概率分布：
$$Q{\mu}(s_t,a_t) = \mathbb{E}\_{r_t,s_{t+1}\sim E}\left[r(s_t,a_t) + \gamma Q^{\mu} (s_{t+1},\mu(s_{t+1}))\right] \tag{3}$$
而第一个期望只和environment相关。这就意味着可以使用off-policy方法学习$Q{\mu}$。
在DQN中，作者使用replay buffer和target network缓解了non-linear funnction approximator不稳定的问题，作者在这篇文章将它们推广到了DDPG上面。

### DDPG
直接将Q-learning推广到continuous action space是不可行的，因为action是continuous的，对其进行max等greedy操作是不可行的。这种优化方法只适合trival action spaces的情况。所以这里使用的是DPG(deterministic policy gradient)，将其推广到non-linear case，DPG是一种actor-critic的方法。
DPG使用一个参数化的actor function $\mu(s|\theta{\mu})$作为当前的policy，它将一个states直接mapping到一个specific action。$Q(s,a)$作为critic使用Q-learning中的Bellman公式进行更新。Actor的更新直接应用chain rule到$J$的expected reutrn ，更新actor的参数如下：
\begin{align\*}
\nabla_{\theta{\mu}} &\approx \mathbb{E}\_{s_t\sim \rho^{\beta} }\left[\nabla_{\theta^{\mu} }Q(s,a|\theta^Q )|\_{s=s_t, a= \mu(s_t|\theta^{\mu} )}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\frac{\partial Q(s,a|\theta^Q )}{\partial\theta^{\mu} }|\_{s=s_t, a= \mu(s_t|\theta^{\mu} )}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\frac{\partial Q(s,a|\theta^Q )}{\partial a}|\_{s=s_t, a= \mu(s_t)}\frac{\partial \mu(s_t|\theta^{\mu} )}{\partial\theta^{\mu} }|\_{s=s_t}\right]\\\\
&= \mathbb{E}\_{s_t\sim \rho{\beta}}\left[\nabla_a Q(s,a|\theta^Q )|\_{s=s_t, a= \mu(s_t)} \nabla_{\theta_{\mu}} \mu(s|\theta_{\mu})|_{s=s_t}\right]\\\\ \tag{4}
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
随机初始化critic 网络$Q(s,a |\theta Q)$，和actor网络$\mu(s|\theta^{\mu} )$的权重$\theta^Q $和$\theta^{\mu} $
初始化target networks　$Q'$和$\mu'$的权重$\theta{Q'}\leftarrow \theta^Q ,\theta^{\mu'} \leftarrow \theta^{\mu} $
初始化replay buffer $R$
**for** episode = 1, M **do**
初始化一个随机process $N$用于exploration
receive initial observation state $s_1$
for $t=1, T$ do
根据behaviour policy选择action $a_t = \mu(s_t| \theta{\mu}) + N_t$
执行action $a_t$，得到$r_t$和$s_{t+1}$
将transition $s_t, a_t, r_t, s_{t+1}$存到$R$
从$R$中采样$N$个transition $s_i, a_i, r_i, s_{i+1}$
设置target value $y_i = r_i + \gamma Q'(s_{i+1}, \mu'(s_{i+1}|\theta{\mu'})|\theta^{Q'} )$
使用$L = \frac{1}{N}\sum_i(y_i-Q(s_i,a_i|\theta Q))^2 $更新critic
使用sampled policy gradient 更新acotr:
$$\nabla_{\theta{\mu}}\approx \frac{1}{N}\sum_i\nabla_a Q(s,a|\theta^Q )|\_{s=s_i, a=\mu(s_i)}\nabla\_{\theta^{\mu} }\mu(s|\theta^{\mu} )|\_{s_i}$$
更新target networks:
$$\theta'\leftarrow \tau \theta + (1-\tau) \theta'$$
end for
end for


### 实验
所有任务中，都使用了low-dimensional state和high-dimensional renderings。在DQN中，为了让问题在high dimensional environment中fully observable，使用了action repeats。在agent的每一个timestep中，进行$3$个timesteps的仿真，包含repeating action以及rendering。因此agent的observation包含$9$个feature maps（RGB，每一个有3个renderings），可以让agent推理不同frames之间的differences。frames进行下采样，得到$64\times 64$的像素矩阵，然后$8$位的RGB值转化为$[0,1]$之间的float points。
在训练的时候，周期性的进行test，test时候的不需要exploration noise。实验表明，去掉不同的组件，即contribution中的几点之后，结果都会比原来差。没有使用target network的话，结果尤其差。
作者使用了两个baselines normalized scores，第一个是naive policy，在action space中均匀的采样action得到的mean return，第二个是iLQG。normalized之后，naive policy的mean score是0，iLQG的mean score是$1$。DDPG能够学习到好的policy，在某些任务上甚至比iLQG还要好。

## 参考文献
Policy Gradient
1.https://arxiv.org/pdf/1509.02971.pdf
2.https://medium.com/@jonathan_hui/rl-policy-gradients-explained-9b13b688b146
3.https://medium.com/@jonathan_hui/rl-policy-gradients-explained-advanced-topic-20c2b81a9a8b

