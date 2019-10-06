---
title: deterministic policy gradient
date: 2019-07-16 10:31:55
tags:
 - pg
 - dpg
 - policy gradient
 - 深度学习
 - 强化学习
categories: 强化学习
mathjax: true
---

Deterministic policy gradient比stochastic policy gradient要好，尤其是high dimensional tasks上。此外，dpg不需要消耗更多的计算资源。还有就是对于一些应用，有可导的policy，但是没有办法加noise，这种情况下dpg更合适。

## 简介
本文主要介绍了deterministic policy gradient算法。它属于policy gradient的一种，只不过是将stochastic policy改成了deterministic policy，和stochastic policy一样，deterministic policy也有相应的deterministic policy gradient theorem。
实际上deterministic policy gradient是action value function的gradient的期望，这让deterministic policy gradient比stochastic policy gradient要更efficiently。还介绍了保证足够的exploration的off-policy actor-critic算法学习determinitic target policy。

## stochastic policy gradient vs deterministic policy gradiet
### sotchastic policy gradient
Policy gradient的basic idea是用参数化的probability distribution $\pi_\theta(a|s) = \mathbb{P}\left[a|s,\theta\right]$表示policy，在state $s$处根据$\theta$表示的policy $\pi$随机选择action $a$。Policy gradient通过对policy 进行采样，根据stochastic policy gradient theorem近似计算$J$相对于参数$\theta$的累计梯度然后调整policy的参数$\theta$朝着使$J$更大的方向移动。

### deterministic policy gradient
用$a=\mu_\theta(s)$表示deterministic policy，根据stochastic policy gradient theorm，很容易想到是否有deterministic policy gardient theorem，即朝着使得cumulative reward更大的方向更新policy的参数。[deterministic policy gradient](https://arxiv.org/pdf/1509.02971.pdf)证明了deterministic policy gradient是存在的，可以使用action value function的梯度近似得到。在某些情况下，deterministic policy gradient还可以看成stochastic policy gradient的特殊情况。

### spg vs dpg
spg和dpg的区别和联系
1. spg和dpg的第一个显著区别就是积分的space是不同的。Spg中policy gradient是在action和state spaces上进行积分的，而dpg的policy gradient仅仅在state space上进行积分。因此，计算spg需要更多samples，尤其是action spaces维度很高的情况下。
2. 为了充分explore整个state和action space，需要使用stochastic policy。而在使用deterministic policy时，为了确保能够持续的进行explore，就需要使用off-policy的算法了，behaviour policy使用stochastic policy进行采样，target policy是deterministic policy。作者使用deterministic policy gradient推导出了一个off-policy的actor-critic方法，使用可导的function approximators估计action values，然后利用这个function的梯度更新policy的参数，同时为了确保policy gradient没有bias，使用而来compatible function。

## stochastic policy gradient theorem
详细的介绍可以查看[policy gradient](http://mxxhcm.github.io/2019/09/07/gradient-method-policy-gradient/)。spg的基本想法就是朝着$J$的梯度方向更新policy的参数。对$J(\pi_{\theta})$对$\theta$求导，得到：
\begin{align\*}
\nabla_{\theta} J(\pi_{\theta})&=\int_S\rho^{\pi} (s) \int_A\nabla_\theta\pi_\theta (a|s)Q^{\pi} (s,a) dads \tag{1}\\\\
&=\mathbb{E}\_{s\sim \rho^{\pi} , a\sim \pi_\theta}\left[\nabla_\theta \log\pi_\theta(a|s)Q^{\pi} (s,a)\right] \tag{2}
\end{align\*}
这就是policy gradient theorem。从这个theorem中，我们得到了一个有用的信息，state distribution $\rho\pi(s)$取决于policy parameters。我们计算$J$关于$\theta$的梯度时，需要计算statedistribution，很难实现，policy gradient theorem消去了对$\rho^{\pi} $的依赖，具有很重要的实用价值。同时利用log derivative trick将performance gradient的估计变成了一个期望，可以通过sampling估计。这个方法中需要使用$Q^{\pi} (s,a)$，估计$Q$不同方法就是不同的算法。比如最简单的使用sample return $G_t$估计$Q^{\pi} (s, a)$，就是REINFORCE算法。

## stochastic actor-critic 算法
Actor-critic是一个基于policy gradient theorem的结构，包含policy和value function两个部分。Actors通过stochastic gradient ascent调整stochastic policy $\pi_\theta(s)$的参数$\theta$；同时critic用一个action-value function $Q^w (s,a)$近似$Q^\pi (s,a)$, $w$是function approximation的参数。Critic一般使用policy evaluation方法进行学习，比如使用td和mc等估计action value function $Q^w (s,a)\approx Q^\pi (s,a)$。一般来说，使用$Q^w (s,a)$代替真实的$Q^\pi (s,a)$会引入bias，但是，如果function approximator是compatible，即满足以下两个条件，就会是无偏的：
1. $Q^w (s,a) = \nabla_\theta \log\pi_\theta(a|s)^T w$
2. 参数$w$最小化mse: $\epsilon^2 (w)=\mathbb{E}\_{s\sim \rho^\pi ,a\sim \pi_\theta}\left[(Q^w (s, a)-Q^\pi (s,a))^2 \right]$，这样就没有bias了，即：
$$\nabla_\theta J(\pi_\theta)=\mathbb{E}\_{s\sim \rho^\pi, a\sim \pi_\theta}\left[\nabla_\theta \log\pi_\theta(a|s)Q^w (s,a)\right] \tag{3}$$

直观上来说，条件1说的是compatible function approximators是stochastic policy梯度$\nabla_\theta \log\pi_\theta(a|s)$的线性features，条件2需要满足$Q^w (s,a)$是$Q^\pi (s,a)$的linear regression soulution。在实际应用中，条件2会放宽，使用如TD之类policy evaluation算法更efficiently的估计value function。事实上，如果条件1和2都满足的话，整个算法等价于没有使用critic，和REINFORCE算法很像。

## off-policy actor critic
在off-policy设置中，performance objective通常改成target policy的value function在behaviour policy的state distribution上进行平均，用$\beta(a|s)$表示behaviour policy：
\begin{align\*}
J_\beta(\pi_\theta) &= \int_S\rho^\beta (s) V^\pi (s)ds\tag{4}\\\\
&=\int_S\int_A\rho^\beta (s)\pi_\theta(a|s)Q^\pi (s,a)dads \tag{5}\\\\
\end{align\*}
对其求导和近似，得到：
\begin{align\*}
\nabla_\theta J_\beta(\pi_\theta) &\approx \int_S\int_A\rho^\beta (s)\nabla_\theta\pi_\theta(a|s) Q^\pi (s,a)dads\tag{6} \\\\
&=\mathbb{E}\_{s\sim \rho\beta, a\sim \beta}\left[\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}\nabla_\theta \log\pi_\theta(a|s) Q^\pi (s,a) \right]\tag{7}
\end{align\*}
这个近似去掉了一项：$\nabla_\theta Q^\pi (s,a)$，已经有人证明去掉这一项之后仍然会收敛到局部最优。Off-policy actor-critic算法使用behaviour policy $\beta$生成trajectories，critic off-policy的从那些trajectories中估计state value function $V^v (s)\approx v^\pi (s)$。actor使用sgd从这些trajectories中off policy的更新policy paramters $\theta$，同时使用TD-error估计$Q^\pi (s,a)$。actor和critic都是用importance sampling ratio $\frac{\pi_\theta(a|s)}{\beta_\theta(a|s)}$。

## action value gradients
绝大多数的model-free rl算法都属于GPI，迭代的policy evaluation和policy improvement。在contious action spaces中，greedy policy improvement是不可行的，因为在每一步都需要计算一个全局最大值。一个替代的方法是让policy朝着$Q$的gradient方向移动，而不是全局最大化$Q$。具体而言，对每一个访问到的state $s$，policy parameters $\theta^{k+1} $的更新正比于$\nabla_{\theta} Q^{ {\mu}^k } (s, \mu_{\theta}(s) )$。每一个state给出一个不同的方向，可以使用state distribution $\rho^{\mu} (s)$求期望，对最终的方向进行平均：
$$\theta^{k+1} = \theta^k + \alpha \mathbb{E}\_{s\sim \rho^{ {\mu}^k } }\left[\nabla_\theta Q^{ {\mu}^k } (s, \mu_{\theta}(s))\right] \tag{8}$$
通过使用chain rule，我们可以看到policy improvement可以被分解成action-value对于action的gradient和policy相对于policy parameters的gradient：
$$\theta{k+1} = \theta^k + \alpha \mathbb{E}\_{s\sim \rho^{ {\mu}^k } }\left[\nabla_{\theta}\mu_{\theta}(s)\nabla_a Q^{ {\mu}^k } (s,a)|\_{a=\mu_\theta(s_0)}\right] \tag{9}$$
按照惯例来说，$\nabla_{\theta}\mu_{\theta}(s)$是一个jacobian matrix，每一列是policy的$dth$ action对$\theta$的gradient $\nabla_\theta\left[\mu_\theta(s)\right]\_d$。如果改变了policy，state distribution　$\rho^{\mu} $也会改变。如果不考虑distribution的变化的话，这个方法是保证收敛的。不过幸运的是，已经有人证明了和sgd一样，有deterministic policy gradient theorem，不需要计算state distributiond的gradient即可更新参数。

## deterministic policy gradient theorem

### 新的术语定义
- $\rho_0(s)$：初始状态分布
- $\rho^{\mu} (s\rightarrow s', k)：在policy $\mu$下从state $s$出发经过$k$步到达$s'$的概率。
- $\rho^{\mu} (s')$：带有折扣因子的状态分布，定义为：
$$\rho^{\mu} (s') = \int_S \sum\_{k=0}^{\infty} \gamma^{k} \rho_0(s) \rho^{\mu} (s\rightarrow s', k)ds\tag{10}$$

### 证明
deterministic policy定义为：$\mu_\theta: S\rightarrow A, \theta \in \mathbb{R}^n $。定义performance objective $J(\mu_\theta) =\mathbb{E}\left[G_0|\mu\right]$，将performance objective写成expectation如下：
\begin{align\*}
J(\mu_\theta) & = \int_S\rho^\mu (s) G_0 ds\tag{11}\\\\
&= \mathbb{E}\_{s\sim \rho^\mu }\left[r(s, \mu_\theta(s))\right] \tag{12}
\end{align\*}
给出deterministic policy gradient theorem：
假设MDP满足以下条件，即$\nabla_\theta\mu_\theta(s)$和$\nabla_a Q^\mu (s,a)$存在，那么deterministic policy gradient存在，
\begin{align\*}
\nabla_\theta J(\mu_\theta) &= \int_S\rho^\mu (s) \nabla_\theta\mu_\theta(s)\nabla_aQ^\mu (s,a)|\_{a=\mu_\theta(s)}ds\tag{13}\\\\
&=\mathbb{E}\_{s\sim \rho^\mu }\left[\nabla_\theta\mu_\theta(s)\nabla_aQ^\mu (s,a)|\_{a=\mu_\theta(s)}\right] \tag{14}
\end{align\*}
证明：
\begin{align\*}
\nabla\_{\theta}V^{\mu} (s) & =  \nabla\_{\theta} Q^{\mu} (s, \mu(s))\tag{15}\\\\
& = \nabla\_{\theta} \left( R(s,\mu(s)) + \int_S \gamma p(s'|s,\mu(s))V^{\mu} (s') ds' \right)\tag{16}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a R(s,a)|\_{a=\mu(s)} + \nabla\_{\theta}\int_S \gamma p(s'|s,\mu(s))V^{\mu} (s') ds'\tag{17}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a R(s,a)|\_{a=\mu(s)} + \int_S \gamma \left(p(s'|s,\mu(s))\nabla\_{\theta} V^{\mu} (s') + \nabla\_{\theta}\mu(s)\nabla_a p(s'|s, a)|\_{a=\mu(s)} V^{\mu} (s') \right) ds'\tag{18}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a \left( R(s,a) + \nabla\_{\theta}\mu(s)\nabla_a p(s'|s, a)|\_{a=\mu(s)} V^{\mu} (s')ds' \right) |\_{a=\mu(s)} + \int_S \gamma p(s'|s,\mu(s))\nabla\_{\theta} V^{\mu} (s')  ds'\tag{19}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a Q^{\mu}(s,a) |\_{a=\mu(s)} + \int_S \gamma p(s\rightarrow s', 1, \mu(s))\nabla\_{\theta} V^{\mu} (s')  ds'\tag{20}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a Q^{\mu}(s,a) |\_{a=\mu(s)}\\\\
&\qquad+ \int_S \gamma p(s\rightarrow s', 1, \mu) \left(\nabla\_{\theta}\mu(s') \nabla_a Q^{\mu} (s', a') |\_{a' = \mu(s')}  + \int_S \gamma p(s' \rightarrow s^{''}, 1, \mu(s') ) \nabla\_{\theta} V^{\mu} (s'') ds^{''} \right) ds' \tag{21}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a Q^{\mu}(s,a) |\_{a=\mu(s)}\\\\
&\qquad + \int_S \gamma p(s\rightarrow s', 1, \mu) \nabla\_{\theta}\mu(s') \nabla_a Q^{\mu} (s', a') |\_{a' = \mu(s')}ds' \\\\
&\qquad + \int_S \gamma p(s\rightarrow s', 1, \mu) \int_S \gamma p(s' \rightarrow s^{''}, 1, \mu(s') )\nabla\_{\theta} V^{\mu} (s')ds^{''} ds' \tag{22}\\\\
& = \nabla\_{\theta}\mu(s) \nabla_a Q^{\mu}(s,a) |\_{a=\mu(s)}\\\\
&\qquad + \int_S \gamma p(s\rightarrow s', 1, \mu) \nabla\_{\theta}\mu(s') \nabla_a Q^{\mu} (s', a') |\_{a' = \mu(s')} ds'\\\\
&\qquad + \int_S \gamma^2 p(s\rightarrow s', 2, \mu)\nabla\_{\theta} V^{\mu} (s') ds' \tag{23}\\\\
& = \int_S \sum\_{t=0}^{\infty} \gamma^t p(s\rightarrow s', t, \mu)\nabla\_{\theta} \mu(s')\nabla\_{a} Q^{\mu} (s', a)|\_{a=\mu(s')} ds' \tag{24}\\\\
\end{align\*}
所以：
\begin{align\*}
\nabla\_{\theta} J(\mu) & = \nabla\_{\theta} \int_S \rho_0(s) V^{\mu} (s) ds \tag{25}\\\\
& = \int_S \rho_0(s) \nabla\_{\theta} V^{\mu} (s) ds \tag{26}\\\\
& = \int_S\int_S \sum\_{t=0}^{\infty} \gamma^t\rho_0(s) p(s\rightarrow s', t, \mu)\nabla\_{\theta} \mu(s')\nabla\_{a} Q^{\mu} (s', a)|\_{a=\mu(s')} ds' ds\tag{27}\\\\
& = \int_S\rho^{\mu} (s) \nabla\_{\theta} \mu(s)\nabla\_{a} Q^{\mu} (s, a)|\_{a=\mu(s)} ds\tag{28}\\\\
& = \int_S\rho^{\mu} (s) \mu(s)\nabla\_{\theta}\log \mu(s)\nabla\_{a} Q^{\mu} (s, a)|\_{a=\mu(s)} ds\tag{29}\\\\
& = \mathbb{E}\_{s\sim \rho^\mu, a\sim \mu(s)}\left[ \nabla\_{\theta}\log \mu(s)\nabla\_{a} Q^{\mu} (s, a) \right]\tag{30}\\\\
\end{align\*}
公式$(25)$到公式$(26)$是因为$\rho_0(s_0)$和$\mu$无关。。




## spg的limit
dpg theorem看起来和spg theorem很不像，事实上，对于一大类stochastic polices来说，dpg事实上是spg的一个特殊情况。如果使用deterministic policy $\mu_\theta:S\rightarrow A$和variance parameter $\sigma$表示某些stochastic policy $\pi_{\mu_{\theta,\sigma}}$，比如$\sigma = 0$时，$\pi_{\mu_{\theta, 0}} \equiv \mu_\theta$，当$\sigma \rightarrow 0$时，stochastic policy gradient收敛于deterministic policy gradient。
考虑一个stochastic policy $\pi_{\mu_{\theta,\sigma}}$让$\pi_{\mu_{\theta,\sigma}}(s,a)=v_\sigma(\mu_\theta(s),a)$，其中$\sigma$是控制方差的参数，并且$v_\sigma$满足条件B.1，以及MDP满足条件A.1和A.2，那么
$$\lim_{\sigma\rightarrow 0}\nabla_\theta J(\pi_{\mu_{\theta, \sigma}}) = \nabla_\theta J(\mu_\theta) \tag{31} $$
其中左边的gradient是标准spg的gradient，右边是dpg的gradient。
这就说明spg的很多方法同样也是适用于dpg的。

### deterministic actor-critic 
接下来使用dpg theorem推到on-policy和off-policy的actor-critic算法。从最简单的on-policy更新，使用Sarsa critic开始，然后考虑off-policy算法，使用Q-learning critic描述核心思想。这些算法在实践中可能会有收敛问题，因为function approximator引入的biases问题，以及off-policy引入的不稳定。然后介绍使用compatiable function approximation的方法以及gradient td learning。

### on-policy deterministic actor-critic

### off-policy deterministic actor-critic

### compatible function approximation

## 参考文献
Policy Gradient
1.https://arxiv.org/pdf/1509.02971.pdf
2.http://www0.cs.ucl.ac.uk/staff/d.silver/web/Publications_files/deterministic-policy-gradients.pdf
