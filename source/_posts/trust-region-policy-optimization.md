---
title: trust region policy optimization
date: 2019-09-08 14:24:37
tags:
 - trpo
 - trust region policy optimization
 - 强化学习
 - reinforcement learning
categories: 强化学习
mathjax: true
---

## Trust Region Policy Optimization
作者提出了optimizing policies的一个迭代算法，理论上保证可以以non-trivial steps单调改善plicy。对经过理论验证的算法做一些近似，产生一个实用算法，叫做Trust Region Policy Optimization(TRPO)。这个算法和natural policy gradient很像，并且在大的非线性网络优化问题上有很高的效率。TRPO有两个变种，single-path方法应用在model-free环境中，vine方法，需要整个system能够能够从特定的states重启，通常在仿真环境中可用。

## 术语定义
更多介绍可以点击查看[reinforcement learning an introduction 第三章]()
1. 状态集合
$\mathcal{S}$是有限states set，包含所有state的可能取值
2. 动作集合
$\mathcal{A}$是有限actions set，包含所有action的可能取值
3. 转换概率矩阵或者状态转换函数
$P:\mathcal{S}\times \mathcal{A}\times \mathcal{S} \rightarrow \mathbb{R}$是transition probability distribution，或者写成$p(s_{t+1}|s_t,a_t)$
4. 奖励函数
$R:\mathcal{S}\times \mathcal{A}\rightarrow \mathbb{R}$是reward function
5. 折扣因子
$\gamma \in (0, 1)$
6. 初始状态分布
$\rho_0$是初始状态$s_0$服从的distribution，$s_0\sim \rho_0$
7. 带折扣因子的MDP
定义为tuple $\left(\mathcal{S},\mathcal{A},P,R,\rho_0, \gamma\right)$
8. 随机策略
选择action，stochastic policy表示为：$\pi_\theta: \mathcal{S}\rightarrow P(\mathcal{A})$，其中$P(\mathcal{A})$是选择$\mathcal{A}$中每个action的概率，$\theta$表示policy的参数，$\pi_\theta(a_t|s_t)$是在$s_t$处取action $a_t$的概率
9. 期望折扣回报
定义
$$G_t = \sum_{k=t}^{\infty} \gamma^{k-t} R_{k+1}$$
为expected discounted returns，表示从$t$时刻开始的expected discounted return；
用
$$\eta(\pi)= \mathbb{E}\_{s_0, a_0, \cdots\sim \pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}\right]$$
表示$t=0$时policy $\pi$的expected discounted return，其中$s_0\sim\rho_0(s_0), a_t\sim\pi(a_t|s_t), s_{t+1}\sim P(s_{t+1}|s_t,a_t)$
10. 状态值函数
state value function的定义是从$t$时刻的$s_t$开始的累计期望折扣奖励：
$$V^{\pi} (s_t) = \mathbb{E}\_{a_{t}, s_{t+1},\cdots\sim \pi}\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \right]$$
或者有时候也定义成从$t=0$开始的expected return：
$$V^{\pi} (s_0) = \mathbb{E}\_{\pi}\left[G_0|S_0=s_0;\pi\right]=\mathbb{E}\_{\pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}|S_0=s_0;\pi \right]$$
11. 动作值函数
action value function定义为从$t$时刻的$s_t, a_t$开始的累计期望折扣奖励：
$$Q^{\pi} (s_t, a_t) = \mathbb{E}\_{s_{t+1}, a_{t+1},\cdots\sim\pi}\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \right]$$
或者有时候也定义为从$t=0$开始的return的期望：
$$Q^{\pi} (s_0, a_0) = \mathbb{E}\_{\pi}\left[G_0|S_0=s_0,A_0=a_0;\pi\right]=\mathbb{E}\_{\pi}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1}|S_0=s_0,A_0=a_0;\pi \right]$$
12. 优势函数
$A^{\pi} (s,a) = Q^{\pi}(s,a) -V^{\pi} (s)$，其中$a_t\sim \pi(a_t|s_t), s_{t+1}\sim P(s_{t+1}|s_t, a_t)$
$V^{\pi} (s)$可以看成状态$s$下所有$Q(s,a)$的期望，而$A^{\pi} (s,a)$可以看成当前的单个$Q(s,a)$是否要比$Q(s,a)$的期望要好，如果为正，说明这个$Q$比$Q$的期望要好，否则就不好。
13. 目标函数
Agents的目标是找到一个policy，最大化从state $s_0$开始的expected return：$J(\pi)=\mathbb{E}\_{\pi} \left[G_0|\pi\right]$，用$p(s\rightarrow s',t,\pi)$表示从$s$经过$t$个timesteps到$s'$的概率，用
$$\rho^{\pi} (s'):=\int_S \sum_{t=0}^{\infty} \gamma^{t} \rho_0(s_0)p(s_0\rightarrow s', t,\pi)ds_0$$
表示$s'$服从的概率分布，其中$\rho_0(s_0)$是初始状态$s_0$服从的概率分布。我们可以将performance objective表示成在state distribution $\rho^\pi $和policy $\pi_\theta$上的期望：
\begin{align\*}
J(\pi_{\theta}) &= \int_S \rho^{\pi} (s) \int_A \pi_{\theta}(s,a) R(s,a)dads\\\\
&= \mathbb{E}\_{s\sim \rho^{\pi} , a\sim \pi_{\theta}}\left[R(s,a)\right] \tag{1}\\\\ 
\end{align\*}
其中$\rho^{\pi} (s)$可以理解为$\rho^{\pi} (s) = P(s_0 = s) +\gamma P(s_1=s) + \gamma^2 P(s_2 = s)+\cdots$，就是policy $\pi$下state $s$出现的概率。这里在每一个$t$处，$s_t=s$都是有一定概率发生的，也就是$\rho_{\pi}(s)$表示的东西。

## Motivation
每一次策略$\pi$的更新，都能使得$\eta(\pi)$单调递增。要是能将它写成old poliy $\pi$和new policy $\hat{\pi}$的关系式就好啦。这里就给出这样一个关系式！恩！就是！
$$\eta(\hat{\pi}) = \eta(\pi) + \mathbb{E}\_{s_0, a_0, \cdots \sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t A_{\pi}(s_t,a_t)\right] \tag{1}$$
将new policy$\hat{\pi}$的期望回报表示为old policy $\pi$的期望回报加上另一项，只要保证这一项是非负的即可。其中$\mathbb{E}\_{s_0, a_0,\cdots, \sim \hat{\pi}}\left[\cdots\right]$表示actions是从$a_t\sim\hat{\pi}(\cdot|s_t)$得到的。
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

## 用期望代替求和
代入$s$的概率分布$\rho_{\pi}(s) = P(s_0 = s) +\gamma P(s_1=s) + \gamma^2 P(s_2 = s)+\cdots, s_0\sim \rho_0$，并将期望换成求和：
\begin{align\*}
\eta(\hat{\pi}) &= \eta(\pi) + \mathbb{E}\_{s_0, a_0, \cdots \sim \hat{\pi}} \left[\sum_{t=0}^{\infty} \gamma^t A^{\pi}(s_t,a_t)\right]\\\\
&=\eta(\pi) +\sum_{t=0}^{\infty} \sum_s P(s_t=s|\hat{\pi}) \sum_a \hat{\pi}(a|s)\gamma^t A^{\pi}(s,a)\\\\
&=\eta(\pi) +\sum_s\sum_{t=0}^{\infty} \gamma^t P(s_t=s|\hat{\pi}) \sum_a \hat{\pi}(a|s)A^{\pi}(s,a)\\\\
&=\eta(\pi) + \sum_s \rho_{\hat{\pi}}(s) \sum_a \hat{\pi}(a|s) A^{\pi} (s,a) \tag{2}\\\\
\end{align\*}
从上面的推导可以看出来，任何从$\pi$到$\hat{\pi}$的更新，只要保证每个state $s$处的expected advantage是非负的，即$\sum_a \hat{\pi}(a|s) A_{\pi}(s,a)\ge 0$，就能说明$\hat{\pi}$要比$\pi$好，在$s$处，新的policy $\hat{\pi}$:
$$\hat{\pi}(s) = arg\ max_a A^{\pi} (s,a) \tag{3}$$
直到所有$s$处的$A^{\pi} (s,a)$为非正停止。当然，在实际应用中，因为各种误差，可能会有一些state的expected advantage是负的。

## $\rho\_{\pi}(s)$近似$\rho\_{\hat{\pi}}(s)$（第一次近似）
上式中包含$\rho_{\hat{\pi}}$，依赖于$\hat{\pi}$，很难直接优化，作者就进行了一个近似：
$$L_{\pi} (\hat{\pi}) = \eta(\pi) + \sum_s\rho_{\pi}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a) \tag{4}$$
$$\eta (\hat{\pi}) = \eta(\pi) + \sum_s\rho_{\hat{\pi}}(s)\sum_a\hat{\pi}(a|s)A^{\pi} (s,a)$$
在$L_{\pi}(\hat{\pi} )$中用$\rho_{\pi}(s)$代替$\rho_{\hat{\pi}}(s)$，从而忽略因为policy改变导致的state访问频率的改变。用$\pi_{\theta}$表示参数化policy，用$\theta$表示$\pi$的参数，如果$\pi(a|s)$是可导的，那么$L_{\pi}(\hat{\pi})$和$\eta(\hat{\pi})$的一阶导相等；此外，当$\hat{\pi} = \pi$时，$L_{\pi}(\hat{\pi}) = \eta(\hat{\pi})$
$$L_{\pi_{\theta_0}} (\pi_{\theta_0}) = \eta(\pi_{\theta_0}) \tag{5}$$
$$\nabla_{\theta} L_{\pi_{\theta_0}}(\pi_{\theta})|\_{\theta=\theta_0} =\nabla_{\theta} \eta(\pi_{\theta})|\_{\theta=\theta_0}\tag{6}$$
证明：
第一个式子不需要证明，而第二个式子，$\pi$相当于已知量，$\hat{\pi}$是关于$\theta$的函数，$\rho_{\hat{\pi}}$是通过样本得到的，不是关于$\hat{\pi}$的函数，最后相当于只有$\hat{\pi}(a|s)$是关于$\hat{\pi}$的函数，所以左右两边就一样了。。（！！！有疑问，就是为什么？$\rho_{\hat{\pi}}$到底是怎么求的，怎么证明）

也就是说在$\pi$处，$L_{\pi}(\pi)$和$\eta(\pi)$是相等的，在$\pi$对应的参数$\theta$周围的无穷小范围内，可以近似认为它们依然相等。$\pi$的参数$\theta_{\pi}$进行足够小的step更新到达新的policy $\hat{\pi}$，相应参数为$\theta_{\hat{\pi}}$，在改进$L_{\pi}$同时也改进了$\eta$，但是这个足够小的step是多少是不知道的。

## 通用随机策略单调增加的证明
有人提出了Conservative policy iteration，该方法提供了$\eta$提升的一个lower bound。用$\pi_{old}$表示current policy，用$\pi' = arg\ min_{\pi'} L_{\pi_{old}}(\pi')$，新的policy $\pi_{new}$定义为：
$$\pi_{new}(a|s) = (1-\alpha) \pi_{old}(a|s)+\alpha\pi'(a|s) \tag{7}$$
有人证明了这样的更新具有以下结果：
$$\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new}) - \frac{2\epsilon \gamma}{(1-\gamma(1-\alpha))(1-\gamma)}\alpha^2 , \epsilon = max_s \vert\mathbb{E}\_{a\sim\pi'}\left[A^{\pi} (s,a)\right]\vert \tag{8}$$
进行缩放：
$$\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new}) - \frac{2\epsilon \gamma}{(1-\gamma)^2 }\alpha^2 \tag{9}$$
在实际应用中，很少使用混合策略，这里使用了一个新的distance measure，叫做total variation divergence，对于离散的概率分布$p,q$来说，定义为：
$$D_{TV}(p||q) = \frac{1}{2} \sum_i \vert p_i -q_i \vert \tag{10}$$
定义$D_{TV}^{max}(\pi, \hat{\pi})$为：
$$D_{TV}^{max} (\pi, \hat{\pi}) = max_s D_{TV}(\pi(\cdot|s) || \hat{\pi}(\cdot|s))\tag{11}$$
让$\alpha = D_{TV}^{max}(\pi_{old}, \pi_{new})$，式子$9$成立。
total variation divergence和KL散度之间的关系：$D_{TV}(p||q)^2 \le D_{KL}(p||q)$，让$D_{KL}^{max}(\pi, \hat{\pi}) = max_s D_{KL}(\pi(\cdot|s)||\hat{\pi}(\cdot|s))$。从公式$9$中可以直接得到：
\begin{align\*}
\eta(\hat{\pi}) &\ge L_{\pi}(\pi_{new}) - \frac{2\epsilon \gamma}{(1-\gamma)^2 }\alpha^2 \\\\
&\ge L_{\pi}(\hat{\pi}) - \frac{2\epsilon \gamma}{(1-\gamma)^2 }D_{KL}^{max}(\pi, \hat{\pi}) \\\\
& \ge L_{\pi}(\hat{\pi}) - CD_{KL}^{max}(\pi, \hat{\pi}), C=\frac{2\epsilon \gamma}{(1-\gamma)^2} \tag{12}
\end{align\*}
根据公式$12$，我们能生成一个单调非递减的sequence：$\eta(\pi_0)\le \eta(\pi_1) \le \eta(\pi_2) \le \cdots$，记$M_i(\pi) = L_{\pi_i}(\pi) - CD_{KL}^{max}(\pi_i, \pi)$，有：
因为：
$$\eta(\pi_{i+1}) \ge M_i(\pi_{i+1})$$
$$\eta(\pi_i) = M_i(\pi_i)$$
上面的第一个式子减去第二个式子得到：
$$\eta(\pi_{i+1}) - \eta(\pi_i)\ge M_i(\pi_{i+1})-M_i(\pi_i) \tag{13}$$
在每一次迭代的时候，最大化$M_i$就能够保证$\eta$是非递减的。这种算法是minorizaiton maximization的一种。$M_i$是miorize $\eta$的近似目标。

## 参数化策略的优化（第二次近似）
前面几小节考虑的optimization问题和$\pi$无关，并且假设所有的states都可以被evaluated。这一节介绍如何在有限的样本下和任意的参数化策略下，从理论基础推导出一个实用的算法。
用$\theta$表示参数化策略$\pi_{\theta}(a|s)$的参数$\theta$，将目标表示成$\theta$而不是$\pi$的函数，即用$\eta(\theta)$表示原来的$\eta(\pi_\theta)$，用$L_{\theta}(\hat{\theta})$表示$L_{\pi_{\theta}}(\pi_{\hat{\theta}})$，用$D_{KL}(\theta||\hat{\theta})$表示$D_{KL}(\pi_{\theta}||\pi_{\hat{\theta}})$。用$\theta_{old}$表示我们想要改进的policy参数。
上一小节我们得到$\eta(\theta) \ge L_{\theta_{old}}(\theta) - CD_{KL}^{max}(\theta_{old}, \theta)$，当$\theta = \theta_{old}$时取等。通过最大化等式右边，可以提高$\eta$的下界：
$$maximize_{\theta}\left[L_{\theta_{old}}(\theta) - CD_{KL}^{max}(\theta_{old}, \theta)\right]\tag{14}$$
在实践中，如果使用penalty coefficient $C$，steps size很小，可以使用new policy 和old policy之间的KL散度约束采取更大的steps，这个约束叫做trust region constraint:
$$maxmize_{\theta} L_{\theta_{old}} (\theta),\qquad s.t. D_{KL}^{max}(\theta_{old},\theta) \le \delta \tag{15}$$
在state space的每一个state都添加一个KL散度进行约束。由于约束太多，这个问题还是不能解，这里使用average KL divergence:
$$\bar{D}\_{KL}^{\rho}(\theta_1, \theta_2) = \mathbb{E}\_{s\sim \rho}\left[D_{KL}(\pi_{\theta_1}(\cdot|s) || \pi_{\theta_2}(\cdot|s))\right] \tag{16}$$
将公式$15$近似成：
$$maxmize_{\theta} L_{\theta_{old}} (\theta), \qquad s.t. \bar{D}\_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta \tag{17}$$

## 目标函数和约束的采样估计（第三次近似）
上一节介绍的是关于policy parameter的有约束优化问题，约束条件为每一次policy更新时限制policy变化的大小，优化expected toral reward $\eta$的一个估计值。这一节使用Monte Carlo采样近似目标和约束函数。
代入$L_{\theta_{old}}$的等式，得到：
$$maxmize_{\theta}\sum_s \rho_{\theta_{old}}(s) \sum_a\pi_{\theta}(a|s)A_{\theta_{old}}(s,a), \qquad s.t. \bar{D}\_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta \tag{17}$$
首先用期望$\frac{1}{1-\gamma}\mathbb{E}\_{s\sim \rho_{\theta_{old}}}\left[\cdots\right]$代替目标函数中的$\sum_s\rho_{\theta_{old}}(s) \left[\cdots\right]$。接下来用$Q$值$Q_{\theta_{old}}$代替advantage $A_{\theta_{old}}$，结果多了一个常数项，不影响。最后使用importance smapling代替actions上的求和。使用$q$表示采样分布，$q$分布中单个的$s_n$对于loss函数的贡献在于：
$$\sum_a \pi_{\theta}(a|s_n) A_{\theta_{old}}(s_n,a) = \mathbb{E}\_{a\sim q}\left[\frac{\pi_{\theta} (a|s_n) }{q(a|s_n)}A_{\theta_{old}}(s_n,a) \right]$$
上面的公式就是使用importance sampling代替求和。将$A$展开：
\begin{align\*}
\sum_a \pi_{\theta}(a|s) A_{\theta_{old}}(s,a) &= \sum_a \pi_{\theta}(a|s)\left( Q_{\theta_{old}}(s,a)  - V_{\theta_{old}}(s)\right)\\\\
&= \sum_a \pi_{\theta}(a|s)Q_{\theta_{old}}(s,a)- \sum_a \pi_{\theta}(a|s)V_{\theta_{old}}(s)\\\\
&= \sum_a \pi_{\theta}(a|s)Q_{\theta_{old}}(s,a)- V_{\theta_{old}}(s)\\\\
\end{align\*}
将公式$17$的优化问题转化为：
$$maxmize_{\theta} \mathbb{E}\_{s\sim\rho_{\theta_{old}}, a\sim q}\left[\frac{\pi_{\theta} (a|s) }{q(a|s)}Q_{\theta_{old}}(s,a)\right] \qquad s.t. \mathbb{E}\_{s\sim \rho_{\theta_{old}}}\left[D_{KL}(\pi_{\theta_{old}}(\cdot|s)||\pi_{\theta}(\cdot|s))\right]\le \delta \tag{18}$$
接下来要做的就是用采样代替期望，用经验估计代替$Q$值。接下来会介绍两种方法进行估计。

第一个叫做single path，通常用在policy gradient estimation，基于单个轨迹的采样。第二个叫做vine，构建一个rollout set，从rollout set的每一个state处执行多个actions。这种方法经常用在policy iteration方法上。

### Single Path
采样$s_0\sim \rho_0$，模拟policy $\pi_{\theta_{old}}$一些timesteps生成一个trajectory $s_0, a_0, s_1, a_1, \cdots, s_{T-1}, a_{T-1}, s_T$，因此$q(a|s) = \pi_{\theta_{old}}(a|s)$。根据trajectory对每一个state action pair $(s_t,a_t)$计算$Q_{\theta_{old}}(s,a)$。

### Vine
采样$s_0\sim \rho_0$，模拟policy $\pi_{\theta_i}$生成一系列trajectories。在这些trajectories选择一个具有$N$个states的子集，表示为$s_1, c\dots, s_N$，这个集合称为rollout set。对于rollout set中的每一个state $s_n$，根据$a_{n,k}\sim q(\cdot|s_n)$采样$K$个actions。任何$q(\cdot|s_n)$都行，在实践中，$q(\cdot|s_n) = \pi_{\theta_i}(\cdot|s_n)$适用于contionous problems，像机器人运动；而均匀分布适用于离散任务，如Atari游戏。
对于$s_n$处的每一个action $a_{n,k}$，从$s_n$和$a_{n,k}$处进行rollout，估计$\hat{Q}\_{\theta_i}(s_n, a_{n,k})$。在小的有限action spaces情况下，我们可以对从给定状态任何可能的action生成一个rollout，单个$s_n$对$L_{\theta_{old}}$的贡献如下：
$$L_n(\theta) = \sum_{k=1}^K \pi_{\theta} (a_k|s_n) \hat{Q}(s_n, a_k)$$
其中action space是$\mathcal{A} = \{a_1, a_2,\cdots, a_K\}$。在大的连续state space中，可以使用importance sampling构建一个新的目标近似。从$s_n$处计算的$L_{\theta_{old}}$的self-normalized 估计是：
$$L_n(\theta) = \frac{\sum_{k=1}^K \frac{\pi_{\theta}(a_{n,k}|s_n)}{\pi_{\theta_{old}}(a_{n,k}|s_n)}\hat{Q}(s_n, a_{n,k}}{\sum_{k=1}^K \frac{\pi_{\theta}(a_{n,k}|s_n)}{\pi_{\theta_{old}}(a_{n,k}|s_n)}}$$
假设在$s_n$处执行了$K$个actions $a_{n,1}, a_{n,2}, \cdots, a_{n,K}$。Self-normalized 估计去掉了$Q$值baseline的需要。在$s_n\sim \rho(\pi)$上做平均，可以得到$L_{\theta_{old}}$和它的gradient的估计。
Vine比single path好的地方在于，给定相同数量的$Q$样本，目标函数的局部估计有更低的方差，也就是vine能更好的估计advantage。Vine的缺点在于，需要执行更多steps的模拟计算相应的advantage。此外，vine方法需要对rollout set 中的每一个state都生成多个trajectories，这就需要整个system可以重置到任意的一个state，而single path算法不需要，可以直接应用在真实的system中。

## 实用算法
使用上面介绍的single path或者vine进行采样，给出两个算法。重复执行以下步骤：
1. 使用single path或者vine算法产生一系列state-action pairs，使用Monte Carlo估计相应的$Q$值；
2. 利用样本计算公式$18$中目标函数和约束函数的估计值
3. 求出有约束优化问题的近似解，更新policy参数$\theta$，使用共轭梯度和line search。

对于$3$来说，计算KL散度的Hessian矩阵而不是协方差矩阵的梯度，生成Fisher information matrix。即使用$\frac{1}{N}\sum_{n=1}^N \frac{\partial^2}{\partial \theta_j}D_{KL}(\pi_{\theta_{old}}(\cdot|s_n)||\pi_{\theta}(\cdot|s_n))$近似$A_{ij}$而不是$\frac{1}{N}\sum_{n=1}^N \frac{\partial}{\partial \theta_i}log(\pi_{\theta}(a_n|s_n))\frac{\partial}{\partial \partial_j}log(\pi_{\theta}(a_n|s_n))$。
在前面介绍的理论和本节介绍的算法之间有以下关联：
1. 理论上优化带有KL散度penalty的目标函数。如果这个penalty系数$\frac{2\epsilon \gamma}{(2-\gamma)^2}$产生很小的step，所以我们想要降低系数。经验上来讲，很难选择一个鲁邦的penalty系数，所以我们使用一个hard constraint而不是一个penalty。
2. $D_{KL}^{max}(\theta_{old}, \theta)$是很难计算和估计的，所以我们对$\bar{D}\_{KL}(\theta_{old}, \theta)$进行约束
3. 本文的理论忽略了advantage function的近似误差。

## 和前面工作的联系
本问中提出的目标函数和一些之前的方法有联系，可以在一个统一的policy update框架下。The natural policy gradient可以看成公式$16$的一个特例：使用$L$的一个linear approximation，和$\bar{D}\_{KL}$的一个二次估计，就有了下面的问题：
$$maximize_{\theta} \left[\nabla_{\theta}L_{\theta_{old}}(\theta)|\_{\theta=\theta_{old}}\cdot (\theta-\theta_{old}) \right] \qquad s.t. \frac{1}{2}(\theta_{old}-\theta)^A(\theta_{old})(\theta_{old} - \theta)\le\delta$$
其中$A(\theta_{old})\_{ij} = \frac{\partial}{\partial\theta_i}\frac{\partial}{\partial \theta_j}\mathbb{E}\_{s\sim \rho_{\pi}}\left[D_{KL}(\pi(\cdot|s, \theta_{old})||\pi(\cdot|s, \theta))\right]\_{\theta=\theta_{old}}$，更新公式是$\theta_{new} = \theta_{old}+\frac{1}{\lambda}A(\theta_{old})^{-1} \nabla_{\theta}L(\theta)|\_{\theta=\theta_{old}}$，其中步长$\frac{1}{\lambda}$可以看成算法参数。这和trpo不同，在每一次更新都有constraint。尽管这个差别很小，实验表明它能改善在更大规模问题上算法的性能。
使用$l2$约束的标准policy gradient如下：
$$maximize_{\theta} \left[\nabla_{\theta} L_{\theta_{old}}(\theta)|\_{\theta=\theta_{old}}\cdot (\theta- \theta_{old}) \qquad s.t. \frac{1}{2}\vert \theta-\theta_{old}\vert^2 \le \delta\right]$$

## 参考文献
Trust Region Policy Optimization
1.http://joschu.net/docs/thesis.pdf
2.https://arxiv.org/pdf/1502.05477.pdf
3.https://medium.com/@jonathan_hui/rl-trust-region-policy-optimization-trpo-explained-a6ee04eeeee9
4.https://medium.com/@jonathan_hui/rl-trust-region-policy-optimization-trpo-part-2-f51e3b2e373a
5.https://people.eecs.berkeley.edu/~pabbeel/cs287-fa09/readings/KakadeLangford-icml2002.pdf
6.https://zhuanlan.zhihu.com/p/26308073
7.https://zhuanlan.zhihu.com/p/60257706
