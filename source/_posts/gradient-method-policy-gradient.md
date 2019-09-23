---
title: gradient method policy gradient
date: 2019-09-22 19:37:52
tags:
 - gradient method
 - policy gradient
 - 强化学习
categories: 强化学习
mathjax: true

---

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
$$\nabla\mathbf{\theta} \approx \alpha \frac{\partial J}{\partial \mathbf{\theta}} \tag{14}$$
其中$\alpha$是正定的step size，按照式子$13$进行更新，可以确保$\theta$收敛到$J$的局部最优值对应的local optimal policy。和value based方法不同，$\mathbf{\theta}$的微小改变只能造成policy和state分布的微小改变。

##### 使用值函数辅助学习policy
本文证明了通过使用满足特定属性的辅助近似值函数，利用之前的experience就可以得到$13$的一个无偏估计。REINFORCE方法也找到了$13$的一个无偏估计，但没有使用辅助值函数，此外它的速度要比使用值函数的方法慢很多。学习一个值函数，并用它取减少方差对快速学习是很重要的。

##### 证明policy iteration收敛性
本文还提出了一种方法证明基于actor-critic和policy-iteration架构方法的收敛性。在这篇文章中，他们只证明了使用通用函数逼近的policy iteration可以收敛到local optimal policy。

### Policy Gradient Therorem
智能体每一步的action由policy $\pi$决定：$\pi(s,a,\mathbf{\theta})=Pr\left[a_t=a|s_t=s,\mathbf{\theta}\right],\forall s\in S, \forall a\in A,\mathbf{\theta}\in \mathbb{R}^l $。假设$\pi$是可导的，即$\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}$存在。为了方便，通常把$\pi(s,a,\mathbf{\theta})$简写为$\pi(s,a)$。有两种方式定义智能体的objective，一种是average reward，一种是从指定状态开始的长期奖励。

#### Average Reward(平均奖励)
平均奖励是根据每一个step的的expected reward $\eta(\pi)$对不同的policy进行排名：
$$\eta(\pi) = lim_{t\rightarrow \infty}\frac{1}{t}\mathbb{E}\left[R_1+R_2+\cdots+R_t|\pi\right] = \int_S \rho^{\pi} (s) \int_A \pi(s,a) R(s,a)dads \tag{15}$$
其中$\rho^{\pi} (s) = lim_{t\rightarrow \infty} Pr\left[s_t=s|s_0,\pi\right]$是策略$\pi$下的station distribution。第一个等号中，$R_t$表示$t$时刻的immediate reward，所以第一个等号表示的是在策略$\pi$下$t$个时间步的imediate reward平均值的期望。
第二个等号，$\rho^{\pi}(s)$是从初始状态$s_0$经过$t$步之后state $s$出现的概率。第一个积分是对$s$积分，相当于求的是$s$的期望；然后对$a$的积分，求的是$s$处各个动作$a$出现概率的期望，所以第二个等式后面求的其实就是每一步$R(s,a)$平均值的期望。其实，把$\rho$换一种写法就容易理解了：$\rho^{\pi} (s) = \int_S \sum_{t=0}^{\infty} \rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0$。
给出一种新的state-action value的定义方式：
$$Q^{\pi} (s,a) = \sum_{t=0}^{\infty} \mathbb{E}\left[R_t - \eta(\pi)|s_0=s,a_0=a,\pi\right], \forall s\in S, a\in A \tag{16}$$
value function定义还和原来一样，但是因为$Q$计算方法变了，所以$V$也跟着变了：
$$V^{\pi} (s) = \mathbb{E}\_{\pi(a';s)}\left[Q^{\pi}(s,a')\right] \tag{17}$$

#### Long-term Accumated Reward from Designated State(从指定状态开始的累计奖励)
第二种情况是指定初始状态$s_0$，我们计算从这个状态开始得到的accumulated reward：
$$\rho(\pi) = \mathbb{E}\left[\sum_{t=0}^{\infty} \gamma^{t-1} R_t|s_0,\pi\right]\tag{18}$$
相应的state-action如下：
$$Q^{\pi} (s,a) = \mathbb{E}\left[\sum_{k=1}^{\infty} R_{t+k}|s_t=s,a_t=a,\pi\right] \tag{19}$$
其中$\gamma\in[0,1]$是折扣因子，只有在episodic任务中才允许取$\gamma=1$。我们定义$\rho^{\pi} (s)$是从开始状态$s_0$执行策略$\pi$遇到的状态的折扣权重之和：
$$\rho^{\pi} (s) = \sum_{t=1}^{\infty} \gamma^t Pr\left[s_t = s|s_0,\pi\right] = \rho^{\pi} (s) = \int_S \sum_{t=0}^{\infty} \gamma^{t} \rho_0(s_0)p(s_0\rightarrow s, t,\pi)ds_0 \tag{20}$$
$\rho^{\pi} $是从$s_0$开始，到$t=\infty$之间的任意时刻所有能到达state $s$的折扣概率之和。

#### Policy Gradient Theorem
对于任何MDP，不论是average reward还是accumulated reward的形式，都有：
$$\frac{\partial \eta}{\partial \mathbf{\theta}} = \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a), \tag{21}$$
证明：
Average Reward:
\begin{align\*}
\nabla V_{\pi}(s) &= \nabla \left[ \sum_a \pi(a|s)Q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a \left[\nabla\pi(a|s)Q_{\pi}(s,a)\right], \\\\
&= \sum_a \left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\nabla Q_{\pi}(s,a)\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\nabla \left[R(s,a)-\eta(\pi)+\sum_{s',r}p(s',r|s,a)V_{\pi}(s')\right]\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\left[-\nabla \eta(\pi)+ \sum_{s',r}p(s',r|s,a) \nabla V_{\pi}(s')\right]\right], \nabla R(s,a) = 0\\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) - \pi(a|s)\nabla \eta(\pi)+ \pi(a|s) \sum_{s',r}p(s',r|s,a) \nabla V_{\pi}(s')\right]\\\\
&= \sum_a\nabla\pi(a|s)Q_{\pi}(s,a) - \sum_a \pi(a|s)\nabla \eta(\pi) + \sum_a \pi(a|s) \sum_{s',r}p(s',r|s,a) \nabla V_{\pi}(s') \\\\
&= \sum_a\nabla\pi(a|s)Q_{\pi}(s,a) -\nabla \eta(\pi)+ \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a)\nabla V_{\pi}(s'), \sum_s\pi(s,a)=1\\\\
\end{align\*}
移项合并同类项得：
$$\nabla \eta(\pi) = \sum_a\nabla\pi(a|s)Q_{\pi}(s,a) + \sum_a\pi(s,a) \sum_{s',r}p(s',r|s,a) \nabla V_{\pi}(s') - \nabla V_{\pi}(s) \tag{22}$$
同时在上式两边对$\rho^{\pi} $进行求和，得到：
\begin{align\*}
\sum_s \rho^{\pi} (s)\nabla \eta(\pi) &= \sum_s \rho^{\pi} (s)\sum_a \nabla\pi(a|s)Q_{\pi}(s,a) \\\\
&\qquad\qquad\qquad + \sum_s \rho^{\pi}(s) \sum_a\pi(a|s) \sum_{s',r}p(s',r|s,a) \nabla V_{\pi}(s')\\\\
&\qquad\qquad\qquad - \sum_s \rho^{\pi} (s)\nabla V_{\pi}(s) \tag{23}\\\\
&= \sum_s \rho^{\pi} (s)\sum_a \nabla\pi(a|s)Q_{\pi}(s,a) + \sum_s \rho^{\pi}(s') \nabla V_{\pi}(s') - \sum_s \rho^{\pi} (s)\nabla V_{\pi}(s) \tag{24}\\\\
&= \sum_s \rho^{\pi} (s)\sum_a \nabla\pi(a|s)Q_{\pi}(s,a) \tag{25}\\\\
&= \nabla \eta(\pi) \tag{26}\\\\
\end{align\*}
式子$22$到式子$23$其实就是$\sum_s \rho^{\pi}(s) \sum_a\pi(a|s) \sum_{s',r}p(s',r|s,a) = \sum_{s'}\rho^{\pi} (s')$，根据$\rho^{\pi} (s)$表示的意义，显然这是成立的。
\begin{align\*}
\end{align\*}

指定初始状态$s_0$:
\begin{align\*}
\nabla V_{\pi}(s) &= \nabla \left[ \sum_a \pi(a|s)Q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a \left[\nabla\pi(a|s)Q_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\nabla Q_{\pi}(s,a)\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\nabla \left[\sum_{s',r}p(s',r|s,a)(R+\gamma V_{\pi}(s'))\right]\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\nabla \left[\sum_{s',r}p(s',r|s,a)R +\sum_{s',r}p(s',r|s,a)\gamma V_{\pi}(s')\right]\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\left[0 +\sum_{s',r}p(s',r|s,a)\gamma\nabla V_{\pi}(s')\right]\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\sum_{s',r}p(s',r|s,a)\gamma \nabla V_{\pi}(s'))\right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\nabla V_{\pi}(s') \right] \\\\
&= \sum_a\left[\nabla\pi(a|s)Q_{\pi}(s,a) + \pi(a|s)\sum_{s'}\gamma p(s'|s,a)\left( \sum_{a'} \nabla\pi(a'|s')Q_{\pi}(s',a') + \pi(a'|s')\sum_{s''}\gamma p(s''|s',a')\nabla V_{\pi}(s''))\right) \right] \tag{27}\\\\
&= \sum_{x\in S}\sum_{k=0}^{\infty} Pr(s\rightarrow x, k,\pi)\sum_a\nabla\pi(a|x)Q_{\pi}(x,a) \tag{28}\\\\
&= \sum_{x\in S}\rho^{\pi} (x)\sum_a\nabla \pi(a|x) Q_{\pi}(x,a) \tag{29}\\\\
\end{align\*}
式子$(28)$中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，对第$(27)$步进行展开以后，从状态$s$开始，在每一个$k$都有可能到达状态$x$，如果不能到，概率为$0$就是了。对于$V_{\pi}(s_0)$，有：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla_{\theta}V_{\pi}(s_0)\\\\
&= \sum_{s\in S}\( \sum_{k=0}^{\infty} Pr(s_0\rightarrow s,k,\pi) \) \sum_a\nabla{\pi}(a|s)Q_{\pi}(s,a)\\\\
&=\sum_{s\in S}\rho(s)\sum_a \nabla{\pi}(a|s)Q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\rho(s')\sum_s\frac{\eta(s)}{\sum_{s'}\eta(s')}\sum_a \nabla{\pi}(a|s)Q_{\pi}(s,a)\\\\
&=\sum_{s'\in S}\rho(s')\sum_s\mu(s)\sum_a \nabla{\pi}(a|s)Q_{\pi}(s,a)\\\\
&\propto \sum_{s\in S}\mu(s)\sum_a\nabla\pi(a|s)Q_{\pi}(s,a) \tag{30}\\\\
\end{align\*}

从这两种情况的证明可以看出来，和$\frac{\partial \rho^{\pi} (s)}{\partial\mathbf{\theta}}$无关：即策略改变对于states distributions没有影响，这非常有利于使用采样来估计梯度。举个例子来说，如果$s$是根据policy $\pi$的从$\rho$中采样得到的，那么$\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a)$就是$\frac{\partial{\rho}}{\partial\mathbf{\theta}}$的一个无边估计。通常$Q^{\pi}(s,a)$也是不知道的，需要估计。一种方法是使用returns，即$G_t = \sum_{k=1}^{\infty} R_{t+k}-\rho(\pi)$或者$R_t = \sum_{k=1}^{\infty} \gamma^{k-1} R_{t+k}$（在指定初始状态条件下），这就是REINFROCE方法。$\nabla\mathbf{\theta}\propto\frac{\partial\pi(s_t,a_t)}{\partial\mathbf{\theta}}R_t\frac{1}{\pi(s_t,a_t)}$,$\frac{1}{\pi(s_t,a_t)}$纠正了$\pi$的oversampling）。

### Policy Gradient with Approximation(使用近似的策略梯度)
因为$Q^{\pi} $是不知道的，我们希望用函数近似式子(20)中的$Q^{\pi} $，大致求出梯度的方向。用$f_w:S\times A \rightarrow \mathbb{R}$表示$Q^{\pi} $的估计值。在策略$\pi$下，更新$w$的值:
$$\Delta w_t\propto \frac{\partial}{\partial w}\left[\hat{Q}^{\pi} (s_t,a_t) - f_w(s_t,a_t)\right]^2 \propto \left[\hat{Q}^{\pi} (s_t,a_t) - f_w(s_t,a_t)\right]\frac{\partial f_w(s_t,a_t)}{\partial w} \tag{31}$$
$\hat{Q}^{\pi} (s_t,a_t)$是$Q^{\pi} (s_t,a_t)$的一个无偏估计，当这样一个过程收敛到local optimum，$Q^{\pi} (s,a)$和$f_w(s,a)$的均方误差最小时：
$$\epsilon(\omega, \pi) = \sum_{s,a}\rho^{\pi} (s)\pi(a|s;\theta)(Q^{\pi} (s,a))^2 - f^{\pi} (s,a;\omega) \tag{32}$$
即导数等于$0$:
$$\sum_s \rho^{\pi} (s)\sum_a\pi(a|s;\theta)\left[Q^{\pi} (s,a) -f_w (s,a;w)\right]\frac{\partial f_w(s,a)}{\partial w}  = 0\tag{33}$$

#### 定理2：Policy Gradient with Approximation Theorem
如果$f_w$的参数$w$满足式子$32$，并且：
$$\frac{\partial f_w(s,a)}{\partial w} = \frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)} = \frac{\partial \log \pi(s,a)}{\partial \mathbf{\theta}}\tag{34}$$
那么使用$f_w(s,a)$计算的gradient和$Q^{\pi} (s,a)$计算的gradient是一样的：
$$\frac{\partial \rho}{\partial \theta} = \sum_s\rho^{\pi} (s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}f_w(s,a)\tag{35}$$

证明：
将式子$33$代入$32$得到：
\begin{align\*}
&\sum_s\rho^{\pi} (s)\sum_a\pi(s,a)\left[Q^{\pi} (s,a) -f_w(s,a)\right]\frac{\partial f_w(s,a)}{\partial w}\\\\
= &\sum_s\rho^{\pi} (s)\sum_a\pi(s,a)\left[Q^{\pi} (s,a) -f_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}\\\\
= &\sum_s\rho^{\pi} (s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\left[Q^{\pi} (s,a) -f_w(s,a)\right] \tag{36}\\\\
= & 0 \\\\
\end{align\*}
将式子$35$带入式子$21$：
\begin{align\*}
\frac{\partial \eta}{\partial \mathbf{\theta}} & = \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a)\\\\
&= \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a) - \sum_s\rho^{\pi} (s)\sum_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\left[Q^{\pi} (s,a) -f_w(s,a)\right]\\\\
&= \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} \left[Q^{\pi} (s,a) - Q^{\pi} (s,a) +f_w(s,a)\right]\\\\
&= \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} f_w(s,a) \tag{37}\\\\
\end{align\*}
得证$\sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a) = \sum_a \rho^{\pi} (s)\sum_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} f_w(s,a) $。

### Application to Deriving Algorithms and Advantages
给定一个参数化的policy，可以利用定理2推导出参数化value function的形式。比如，考虑在features上进行线性组合的Gibbs分布构成的policy：
$$\pi(a|s) = \frac{e\^{\theta^T \phi_{sa} } }{\sum_b e\^{\theta^T \phi_{sb} }} , \forall s \in S, \forall a \in A \tag{38}$$
其中$\phi_{s,a}$是state-action pair $s,a$的特征向量。满足式子$(28)$的公式如下：
$$\frac{\partial f_w(s,a)}{\partial w} = \frac{\partial \pi(a|s)}{\partial \theta}\frac{1}{\pi(a|s)} = \phi_{sa} - \sum_b\pi(b|s)\phi_{sb}\tag{39}$$
所以：
$$f_w(s,a) = w^T \left[\phi_{sa} - \sum_b\pi(b|s)\phi_{sb} \right]\tag{40}$$
也就是说，$f_w$和policy $\pi$都是feature的线性组合，只不过每一个state处$f_w$的均值都为$0$，$\sum_a\pi(a|s)f_w(s,a) = 0,\forall s\in S$。所以，其实我们可以认为$f_w$是对advantage function $A^{\pi} (s,a) = Q^{\pi} (s,a)- V^{\pi} (s)$而不是$Q^{\pi} (s,a)$的一个近似。式子$(28)$中$f_w$其实是一个相对值而不是一个绝对值。事实上，他们都可对以推广变成一个function加上一个value function。比如式子$(29)$可以变成$\frac{\partial\eta}{\partial \theta} = \sum_s\rho^{\pi}(s) \sum_a \frac{\partial \pi(a|s)}{\partial \theta}\left[f_w(s,a) + v(s)\right]$，其中$v$是一个function，$v$的选择不影响理论结果，但是会影响近似梯度的方差。

### Convergence of Policy Iteration with Function Approximation(使用函数近似的策略迭代的收敛性)

#### 定理3：Policy Iteration with Function Approximation
用$\pi$和$f_w$表示policy和value function的可导函数，并且满足式子$(28)$。$\max_{\theta,s,a,i,j} \vert\frac{\partial^2 \pi(a|s)}{\partial\theta_i \partial\theta_j} \vert\lt B\lt \infty$，假设$\left[\alpha_k\right]\_{k=0}^{\infty}$是步长sequence，$\lim\_{k\rightarrow \infty}\alpha_k = 0$，$\sum_k \alpha_k = \infty$。对于任何有界rewards的MDP来说，任意$\theta_0$，$\pi_k=\pi(\cdot, \theta_k)$定义的$\left[\eta(\pi_k)\right]\_{k=0}^{\infty}$，并且$w_k = w$满足：
$$\sum_s\rho^{\pi_k} (s) \sum_a\pi_k(a|s)\left[Q^{\pi_k} (s,a)-f_w(s,a) \right]\frac{\partial f_w(s,a)}{\partial w}=0 \tag{41}$$
$$\theta_{k+1} = \theta_k + \alpha_k \sum_s\rho^{\pi_k}(s) \sum_a\frac{\partial\pi_k(s,a)}{\partial \theta}f_{w_k}(s,a) \tag{42}$$
一定收敛：$\lim_{k\rightarrow \infty}\frac{\partial \rho(\pi_k)}{\partial \theta} = 0$。


