---
title: gradient method policy gradient
date: 2019-09-07 19:37:52
tags:
 - gradient method
 - policy gradient
 - 强化学习
categories: 强化学习
mathjax: true
---

## Policy Gradient
强化学习有三种常用的方法，第一种是基于值函数的，第二种是policy gradient，第三种是derivative-free的方法，即不利用导数的方法。基于值函数的方法在理论上证明是很难的。这篇论文提出了policy gradient的方法，直接用函数去表示策略，根据expected reward对策略参数的梯度进行更新，REINFORCE和actor-critic都是policy gradient的方法。
本文给出了policy gradient theorem的证明，使用近似的action-value function或者advantage函数，梯度可以表示成experience的估计。同时证明了任意可导的函数表示的policy通过policy iteration都可以收敛到locl optimal policy。

### 值函数方法的缺点
基于值函数的方法，在估计出值函数之后，每次通过greedy算法选择action。这种方法有两个缺点。
- 基于值函数的方法会找到一个deterministic的策略，但是很多时候optimal policy可能是stochastic的。
- 某个action的估计值函数稍微改变一点就可能导致这个动作被选中或者不被选中，这种不连续是保证值函数收敛的一大障碍。

### 用函数表示stochastic policy
Policy gradient用函数表示stochastic policy。比如用神经网络表示的一个policy，输入是state，输出是每个action选择的概率，神经网络的参数是policy的参数。用$\mathbf{\theta}$表示policy参数，用$J$表示该策略的performance measure。然后参数$\mathbf{\theta}$的更新正比于以下梯度：
$$\nabla\mathbf{\theta} \approx \alpha \frac{\partial J}{\partial \mathbf{\theta}} \tag{1}$$
其中$\alpha$是正定的step size，按照式子$(1)$进行更新，可以确保$\theta$收敛到$J$的局部最优值对应的local optimal policy。和value based方法相比，$\mathbf{\theta}$的微小改变只能造成policy和state分布的微小改变。

### 使用值函数辅助学习policy
使用满足特定属性的辅助近似值函数，利用之前的experience就可以得到式子$(1)$的一个无偏估计。REINFORCE方法也找到了式子$(1)$的一个无偏估计，但没有使用辅助值函数，此外它的速度要比使用值函数的方法慢很多。学习一个值函数，并用它取减少方差对快速学习是很重要的。

### 证明policy iteration收敛性
本文还证明了基于actor-critic和policy-iteration架构方法的收敛性。在这篇文章中，他们只证明了使用通用函数逼近的policy iteration可以收敛到local optimal policy。

## Objective Function
智能体每一步的action由policy $\pi$决定：$\pi(s,a,\mathbf{\theta})=Pr\left[a\_t=a|s\_t=s,\mathbf{\theta}\right],\forall s\in S, \forall a\in A,\mathbf{\theta}\in \mathbb{R}^l $。为了方便，通常把$\pi(s,a,\mathbf{\theta})$简写为$\pi(s,a)$。假设$\pi$是可导的，即$\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}$存在。有三种方式定义智能体的objective：
- 计算policy $\pi$下从初始状态$s_0$开始的accumulated reward：
$$J(\theta) = V^{\pi}(s_0) = \mathbb{E}\_{\pi}\left[G_0\right] = \mathbb{E}\_{\pi} \left[\sum\_{t=0}^{\infty} \gamma^{t-1} R_t | s_0 \right] \tag{2}$$
其中$0 \le \gamma \le 1$，在continuing case中，$0 \le \gamma \lt 1$，而在episodic情况下，$\gamma$能取到$1$，$0 \le \gamma \le 1$。
- 计算policy $\pi$每个timestep的immediate reward的均值，即average reward。
$$J(\theta) = \mathbb{E}\_{\pi}\left[R(s, a)\right] = \sum_s d(s) \sum_a\pi(s, a)R(s,a) \tag{3}$$
在connuting problems情况下，没有episode boundaries，如果不使用discount factor，可以这么做；事实上，我觉得在episodic情况下，也能这么做。
- 当没有明确的初始状态时，计算policy $\pi$下所有state value的均值：
$$J(\theta) = \sum_s d(s) V^{\pi} (s) = \sum_s d(s) \sum_a\pi(s, a) Q^{\pi} (s, a) = \mathbb{E}\_{\pi}\left[Q^{\pi}(s, a)\right] \tag{4}$$

其中$R(s,a) = \mathbb{E}\left[R_{t+1}|s_t=s, a_t=a\right]$，$d (s) = lim\_{t\rightarrow \infty} Pr\left[s\_t=s|s\_0,\pi\right]$是策略$\pi$下的stationary distribution。[Stationary distribution](http://mxxhcm.github.io/2019/07/31/markov-matrices/)的意思是就是不论初始状态是什么，经过很多步之后，都会达到一个stable state。其实这三个目标本质上都是一样的，都是要最大化每个时刻agent得到的reward。

### Accumated Reward from Designated State(从指定状态开始的累计奖励)
我们可以指定一个初始状态$s\_0$，计算从这个初始状态开始得到的accumulated reward：
$$\eta(\pi) = V^{\pi} (s_0) = \mathbb{E}\_{\pi}\left[\sum\_{t=0}^{\infty} \gamma^{t-1} R\_t|s\_0\right] = \mathbb{E}\_{\pi}\left[G_0 \right]\tag{5}$$
定义return $G_t$如下：
$$ G_t = \sum\_{k=0}^{\infty} R\_{t+k+1} \tag{8}$$
定义state-action value function和state value function如下：
$$Q^{\pi} (s,a) = \mathbb{E}\_{\pi}\left[G_t|s_t=s, a_t=a\right] = \mathbb{E}\_{\pi}\left[\sum\_{k=1}^{\infty} R\_{t+k}|s\_t=s,a\_t=a\right] \tag{6}$$
$$V^{\pi} (s) = \mathbb{E}\_{\pi}\left[G_t|s_t=s\right] = \mathbb{E}\_{\pi}\left[\sum\_{k=1}^{\infty} R\_{t+k}|s\_t=s\right] \tag{7}$$
其中$\gamma\in[0,1]$是折扣因子，只有在episodic任务中才允许取$\gamma=1$。它们之间的关系如下：
$$ V^{\pi} (s) = \mathbb{E}\_{\pi}\left[Q(s,a)\right] = \sum_a \pi(a|s) Q^{\pi} (s,a) \tag{9}$$
定义$\rho^{\pi} (s)$是从指定的初始状态$s\_0$开始，执行策略$\pi$在$t=\infty$之间的任意时刻所有能到达state $s$的折扣概率之和：
$$\rho^{\pi} (s) = \sum\_{t=1}^{\infty} \gamma^t Pr\left[s\_t = s|s\_0,\pi\right]  =  \sum\_{t=0}^{\infty} \gamma^{t} p(s\_0\rightarrow s, t,\pi) \tag{10}$$
把$\rho^{\pi} (s) $换一种写法就容易理解了：
$$\rho^{\pi} (s) = P(s\_0 = s) +\gamma P(s\_1=s) + \gamma^2 P(s\_2 = s)+\cdots \tag{11}$$

### Average Reward(平均奖励)
Average reward是根据每一个step的的expected reward $\eta(\pi)$对不同的policy进行排名：
$$\eta(\pi) = lim\_{t\rightarrow \infty}\frac{1}{t}\mathbb{E}\left[R\_1+R\_2+\cdots+R\_t|\pi\right] = \int\_S d(s) \int\_A \pi(s,a) R(s,a)dads \tag{12}$$
第一个等号中，$R\_t$表示$t$时刻的immediate reward，所以第一个等号表示的是在策略$\pi$下$t$个时间步的imediate reward平均值的期望。第二个等号后，第一个积分是对$s$积分，相当于求的是$s$的期望；然后对$a$的积分，求的是每一个$s$处对应各个动作$a$出现概率的期望，所以第二个等式后面求的其实就是每一步$R(s,a)$平均值的期望。
Return的定义和accumulated reward不同：
$$G_t = \sum\_{t=0}^{\infty} \left[R_{t+1} - \eta(\pi)\right] \tag{13}$$
因为$G_t$不同，State-action value $Q^{\pi} (s,a)$以及state value $V^{\pi} (s)$的定义和accumulated reward也就不同了：
$$Q^{\pi} (s,a) = \mathbb{E}\_{\pi}\left[G_t|s_t=s, a_t=a\right] = \mathbb{E}\_{\pi}\left[\sum\_{k=1}^{\infty} R\_{t+k}|s\_t=s,a\_t=a\right] $$
$$V^{\pi} (s) = \mathbb{E}\_{\pi}\left[G_t|s_t=s\right] = \mathbb{E}\_{\pi}\left[\sum\_{k=1}^{\infty} R\_{t+k}|s\_t=s\right] $$

### State value的均值
这个和上面的accumulated reward有一定关联，accumulated计算的是$V^{\pi} (s_0)$，而这里计算的是$V^{\pi} (s)$的期望（均值）：
$$ \eta(\pi) = \sum_s d(s) V^{\pi} (s) \tag{14}$$
State action value function和state value function的定义和accumulated reward一样。
定义$\rho^{\pi} $为从任意初始状态$s\_0$经过$t$步之后state $s$出现的概率：
$$\rho^{\pi} (s) =\int_S \sum\_{t=0}^{\infty} \gamma^t \rho_0^{\pi} (s_0) Pr\left[s\_t = s|s\_0,\pi\right] ds_0  = \int\_S \sum\_{t=0}^{\infty} \gamma^{t} \rho\_0^{\pi} (s\_0)p(s\_0\rightarrow s, t,\pi)ds\_0 \tag{15}$$


### Policy Gradient
对于单步的MDP，从分布$\rho^{\pi} (s)$中采样得到$s$，采取action $a$，得到immediate reward $R=R(s,a)$，结束。上面三种目标函数是一样的：
\begin{align\*}
J(\theta) & = \mathbb{E}\_{\pi}\left[R\right]\\\\
& = \sum_s d(s) \sum_a \pi(s,a) R(s,a) \tag{16}\\\\
\end{align\*}
求导有问题！！！！怎么求导得到的。。。
\begin{align\*}
\nabla_{\theta} J(\theta) & = \sum_s d(s) \sum_a \nabla_{\theta}\pi(s,a) R(s,a)\\\\
& = \sum_s d(s) \sum_a\pi(s,a) \nabla_{\theta}\log \pi(s,a) R(s,a)\\\\
& = \mathbb{E}\_{\pi}\left[\nabla_{\theta}\log \pi(s,a) R(s,a)\right] \tag{17}\\\\
\end{align\*}
对于多步的MDP，只需要将$R$换成$Q^{\pi} (s, a)$就行了，上面三种目标函数最后都能够得到：
$$\nabla\_{\theta} J(\theta) = \sum_s d(s) \sum_a\pi(a|s) \nabla\_{\theta} \log\pi(s,a) Q^{\pi} (s,a) = \mathbb{E}\_{\pi} \left[\nabla\_{\theta} \log\pi(s,a) Q^{\pi} (s,a)\right] \tag{18}$$
其中$Q$是根据不同的目标函数定义的state-action value function，目标函数不同，$Q$定义也不同。在其他论文中，$\nabla_{\theta} \log\pi_{\theta}(s,a)$不变，可以把$Q$换成其他目标函数，GAE这篇论文对不同的目标函数进行了总结。

## Policy Gradient Theorem
对于任何MDP，不论是average reward还是accumulated reward的形式，都有：
\begin{align\*}
\nabla_{\theta} \eta & = \sum\_s \rho^{\pi} (s)\sum\_a{\nabla_{\theta}\pi(s,a)}Q^{\pi} (s,a), \tag{19}\\\\
& = \sum\_s \rho^{\pi} (s)\sum\_a{\pi(s,a)\nabla_{\theta}\log\pi(s,a)}Q^{\pi} (s,a), \tag{20}\\\\
& = \mathbb{E}\_{\pi}\left[\nabla_{\theta}\log\pi(s,a)Q^{\pi} (s,a)\right], \tag{21}\\\\
\end{align\*}
证明：
\begin{align\*}
\nabla V\_{\pi}(s) &= \nabla \left[ \sum\_a \pi(a|s)Q\_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum\_a \left[\nabla\pi(a|s)Q\_{\pi}(s,a)\right], \forall s\in S \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\nabla Q\_{\pi}(s,a)\right] \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\nabla \left[\sum\_{s',r}p(s',r|s,a)(R+\gamma V\_{\pi}(s'))\right]\right] \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\nabla \left[\sum\_{s',r}p(s',r|s,a)R +\sum\_{s',r}p(s',r|s,a)\gamma V\_{\pi}(s')\right]\right] \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\left[0 +\sum\_{s',r}p(s',r|s,a)\gamma\nabla V\_{\pi}(s')\right]\right] \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\sum\_{s',r}p(s',r|s,a)\gamma \nabla V\_{\pi}(s'))\right] \\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\sum\_{s'}\gamma p(s'|s,a)\nabla V\_{\pi}(s') \right] \\\\
&= \sum\_a\\\\
&\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\sum\_{s'}\gamma p(s'|s,a)\left( \sum\_{a'} \nabla\pi(a'|s')Q\_{\pi}(s',a') + \pi(a'|s')\sum\_{s''}\gamma p(s''|s',a')\nabla V\_{\pi}(s''))\right) \right] \tag{22}\\\\
&= \sum\_{x\in S}\sum\_{k=0}^{\infty} Pr(s\rightarrow x, k,\pi)\sum\_a\nabla\pi(a|x)Q\_{\pi}(x,a) \tag{23}\\\\
&= \sum\_{x\in S}\rho^{\pi} (x)\sum\_a\nabla \pi(a|x) Q\_{\pi}(x,a) \tag{24}\\\\
\end{align\*}

式子$(15)$中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，对第$(14)$步进行展开以后，从状态$s$开始，在每一个$k$都有可能到达状态$x$，如果不能到$x$，概率为$0$就是了。

### 指定初始状态$s\_0$的accumulated reward
证明思路，在上面我们已经求得了$V^{\pi} (s)$的梯度，而accumulated reward其实就是$V^{\pi} (s_0)$，取$J(\mathbf{\theta}) = V\_{\pi}(s\_0)$，则：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla\_{\theta}V\_{\pi}(s\_0)\\\\
&= \sum\_{s\in S}\( \sum\_{k=0}^{\infty} Pr(s\_0\rightarrow s,k,\pi) \) \sum\_a\nabla{\pi}(a|s)Q\_{\pi}(s,a)\qquad\qquad\qquad;\rho(s) = \sum\_{k=0}^{\infty} Pr(s\_0\rightarrow s,k,\pi) \tag{25}\\\\
&=\sum\_{s\in S}\rho(s)\sum\_a \nabla{\pi}(a|s)Q\_{\pi}(s,a)\tag{26}\\\\
&=\sum\_{s'\in S}\rho(s')\sum\_s\frac{\rho(s)}{\sum\_{s'}\rho(s')}\sum\_a \nabla{\pi}(a|s)Q\_{\pi}(s,a) \qquad\qquad\qquad\qquad; \text{normalize } \rho(s) \tag{27}\\\\
&=\sum\_{s'\in S}\rho(s')\sum\_sd(s)\sum\_a \nabla{\pi}(a|s)Q\_{\pi}(s,a) \tag{28} \qquad\qquad\qquad\qquad\qquad; d(s) = \frac{\rho(s) }{\sum_{s'} \rho(s')} \text{ is stationary distribution}\\\\
&\propto \sum\_{s\in S}d(s)\sum\_a\nabla\pi(a|s)Q\_{\pi}(s,a)\tag{29},\qquad\qquad\qquad\qquad\qquad\qquad\qquad; \sum_s\rho(s)\text{是常数}\\\\
& = \sum\_{s\in S}d(s)\sum\_a\pi(s, a)\nabla\log\pi(a|s)Q\_{\pi}(s,a) \tag{30}\\\\
& = \mathbb{E}\_{\pi}\left[\nabla\log\pi(a|s)Q\_{\pi}(s,a)\right] \tag{31}\\\\
\end{align\*}
其中$\mathbb{E}\_{\pi}$表示$\mathbb{E}\_{s\sim d_{\pi}, a\sim \pi}$，即state和action distributions都遵守policy $\pi$。

### Average Reward:
证明思路，首先求$V$的梯度，带入$Q, V$和$\eta$的关系进行转换，能够得到$\eta$的梯度和$Q,V$梯度之间的关系，最后进行一系列化简即可。
\begin{align\*}
\nabla V\_{\pi}(s) &= \nabla \left[ \sum\_a \pi(a|s)Q\_{\pi}(s,a)\right], \forall s\in S \tag{32}\\\\
&= \sum\_a \left[\nabla\pi(a|s)Q\_{\pi}(s,a)\right], \tag{33} \\\\
&= \sum\_a \left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\nabla Q\_{\pi}(s,a)\right] \tag{34}\\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\nabla \left[\sum\_{s',r}p(s',r|s,a)\left(R(s,a)-\eta(\pi)+V\_{\pi}(s')\right)\right]\right] \tag{35}\\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) + \pi(a|s)\left[-\nabla \eta(\pi)+ \sum\_{s',r}p(s',r|s,a) \nabla V\_{\pi}(s')\right]\right], \nabla R(s,a) = 0\tag{36}\\\\
&= \sum\_a\left[\nabla\pi(a|s)Q\_{\pi}(s,a) - \pi(a|s)\nabla \eta(\pi)+ \pi(a|s) \sum\_{s',r}p(s',r|s,a) \nabla V\_{\pi}(s')\right]\tag{37}\\\\
&= \sum\_a\nabla\pi(a|s)Q\_{\pi}(s,a) - \sum\_a \pi(a|s)\nabla \eta(\pi) + \sum\_a \pi(a|s) \sum\_{s',r}p(s',r|s,a) \nabla V\_{\pi}(s') \tag{38}\\\\
&= \sum\_a\nabla\pi(a|s)Q\_{\pi}(s,a) -\nabla \eta(\pi)+ \sum\_a \pi(a|s)\sum\_{s',r}p(s',r|s,a)\nabla V\_{\pi}(s'), \sum\_s\pi(s,a)=1\tag{39}\\\\
\end{align\*}
移项合并同类项得：
$$\nabla \eta(\pi) = \sum\_a\nabla\pi(a|s)Q\_{\pi}(s,a) + \sum\_a\pi(s,a) \sum\_{s',r}p(s',r|s,a) \nabla V\_{\pi}(s') - \nabla V\_{\pi}(s) \tag{40}$$
同时在上式两边对$d(s)$进行求和，得到：
\begin{align\*}
\sum\_s d(s)\nabla \eta(\pi) &= \sum\_s d(s)\sum\_a \nabla\pi(a|s)Q\_{\pi}(s,a) \\\\
&\qquad\qquad\qquad + \sum\_s d(s) \sum\_a\pi(a|s) \sum\_{s',r}p(s',r|s,a) \nabla V\_{\pi}(s')\\\\
&\qquad\qquad\qquad - \sum\_s d(s)\nabla V\_{\pi}(s) \tag{41}\\\\
\nabla \eta(\pi) &= \sum\_s d(s)\sum\_a \nabla\pi(a|s)Q\_{\pi}(s,a) + \sum\_s d(s') \nabla V\_{\pi}(s') - \sum\_s d(s)\nabla V\_{\pi}(s) \tag{42}\\\\
&= \sum\_s d(s)\sum\_a \nabla\pi(a|s)Q\_{\pi}(s,a) \tag{43}\\\\
&= \sum\_s d(s)\sum\_a \pi(s,a) \nabla\log\pi(a|s)Q\_{\pi}(s,a) \tag{44}\\\\
& = \mathbb{E}\_{\pi}\left[\nabla_{\theta}\log\pi(s,a) Q\_{\pi}(s,a)\right] \tag{45}\\\\
\end{align\*}
式子$10$到式子$11$其实就是$\sum\_s d(s) \sum\_a\pi(a|s) \sum\_{s',r}p(s',r|s,a) = \sum\_{s'}d(s')$，根据$\rho^{\pi} (s)$表示的意义，显然这是成立的。
\begin{align\*}
\end{align\*}

### State value的期望
还不会证明。

### 结论
从这两种情况的证明可以看出来，policy gradient和$\frac{\partial \rho^{\pi} (s)}{\partial\mathbf{\theta}}$无关：即可以通过计算，让policy的改变不影响states distributions，这非常有利于使用采样来估计梯度。举个例子来说，如果$s$是根据policy $\pi$的从$\rho$中采样得到的，那么$\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a)$就是$\frac{\partial{\rho}}{\partial\mathbf{\theta}}$的一个无偏估计。通常$Q^{\pi}(s,a)$也是不知道的，需要估计。一种方法是使用returns近似，即$G\_t = \sum\_{k=1}^{\infty} R\_{t+k}-\rho(\pi)$或者$R\_t = \sum\_{k=1}^{\infty} \gamma^{k-1} R\_{t+k}$（在指定初始状态条件下），这就是REINFROCE方法。$\nabla\mathbf{\theta}\propto\frac{\partial\pi(s\_t,a\_t)}{\partial\mathbf{\theta}}R\_t\frac{1}{\pi(s\_t,a\_t)}$,$\frac{1}{\pi(s\_t,a\_t)}$纠正了$\pi$的oversampling）。


## 另一种policy gradient的方法
目标函数$J$如下：
\begin{align\*}
J(\theta) & = \mathbb{E}\_{\tau \sim \pi\_{\theta}(\tau)} \left[R(\tau)\right] \tag{46}\\\\
& = \mathbb{E}\_{\tau \sim \pi\_{\theta}(\tau)} \left[\sum_t R(s_t, a_t)\right] \tag{47}\\\\
& = \int \pi\_{\theta}(\tau) R(\tau) d\tau \tag{48}\\\\
& = \int \pi\_{\theta}(\tau)\sum_t R(s_t, a_t) d\tau \tag{49}\\\\
& \approx \frac{1}{N}\sum_i \sum_t R(s_{i,t}, a_{i,t}) \tag{50}\\\\
\end{align\*}
其中$\tau = s\_0, a\_0, s\_1, a\_1,\cdots \sim \pi\_{\theta}$表示一个episode的trajectory，$R(\tau)$表示这个trajectory的returns($G\_0$)。Policy gradient变成下式，（为什么？？？还是不懂！！CS294上面的推导！！！为什么$\nabla$可以写进积分号里面，$R(\tau)$不也和$\pi_{\theta}$有关？？？）
\begin{align\*}
\nabla\_{\theta}J(\theta) & = \int \nabla\_{\theta} \pi\_{\theta}(\tau) R(\tau)d\tau\\\\
& = \int \pi\_{\theta}(\tau) \nabla\_{\theta}\log\pi\_{\theta}(\tau) R(\tau)d\tau\\\\
& = \mathbf{E}\_{\tau\sim \pi\_{\theta}(\tau)} \left[\nabla\_{\theta} \log\pi\_{\theta}(\tau) R(\tau) d\tau\right] \tag{51}
\end{align\*}
可以将policy gradient表示成期望的形式，然后就可以采样进行估计。对$R(\tau)$进行采样，但是不进行求导。Returns不直接受$\pi\_{\theta}$的影响，$\tau$受$\pi\_{\theta}$的影响，下面是$\log\pi(\tau)$的偏导数计算。
$\pi(\tau)$定义为：
$$\pi\_{\theta}(s\_0,a\_0,\cdots, s\_T,a\_T) = p(s\_0) \prod\_{t=0}^T \pi\_{\theta}(a\_t|s\_t)p(s\_{t+1}|s\_t,a\_t) \tag{52}$$
取$\log$：
$$\log\pi\_{\theta}(s\_0,a\_0,\cdots, s\_T,a\_T) = \log p(s\_0) + \sum\_{t=0}^T\log \pi\_{\theta}(a\_t|s\_t) + \log p(s\_{t+1}|s\_t,a\_t) \tag{53}$$
对$\theta$求偏导，得到：
$$\nabla_{\theta} \log\pi(\tau) = \nabla\_{\theta}\left[\sum\_{t=0}^T \log\pi\_{\theta}(a\_t|s\_t) \right] = \left[\sum\_{t=0}^T \nabla\_{\theta} \log\pi\_{\theta}(a\_t|s\_t) \right]\tag{54}$$
所以，policy gradient：
\begin{align\*}
\nabla\_{\theta}J(\theta) &= \mathbb{E}\_{\tau \sim \pi\_{\theta}(\tau)}\left[\nabla\_{\theta}\log\pi\_{\theta}(\tau) R(\tau) \right]\tag{55}\\\\
& = \mathbb{E}\_{\tau \sim \pi\_{\theta}(\tau)} \left[ \left(\sum\_{t=1}^T\nabla\_{\theta} \log\pi\_{\theta}(a\_{i,t}|s\_{i,t})\right) \left(\sum\_{t=1}^T R(s\_{i,t}, a\_{i,t})\right) \right] \tag{56}\\\\
& \approx \frac{1}{N}\sum\_{i=1}^N \left(\sum\_{t=1}^T\nabla\_{\theta} \log\pi\_{\theta}(a\_{i,t}|s\_{i,t})\right) \left(\sum\_{t=1}^T R(s\_{i,t}, a\_{i,t})\right)\tag{57}\\\\
\end{align\*}
$$ \theta \leftarrow \theta + \alpha \nabla\_{\theta} J(\theta)\tag{58}$$
即用多个trajectories近似计算policy gradietn，更新$\theta$。

### Intution
$\nabla\_{\theta} \log\pi\_{\theta}(a\_{i,t}|s\_{i,t})$是最大对数似然，表示的是对应的trajectory在当前的policy下发生的可能性。将它和returns相乘，如果产生high positive reward，增加policy的可能性，如果是high negetive reward，减少policy的可能性。在一个trajectory中的states具有很强的相关性，这个trajectory发生的概率定义为：
$$\pi(\tau) = p(s\_0) \prod\_{t=0}^T \pi\_{\theta}(a\_t|s\_t)p(s\_{t+1}|s\_t,a\_t) \tag{59}$$
但是连续的乘法可能会产生梯度消失或者梯度爆炸问题。policy gradient将连乘变成了连加。
### Policy Gradient with Monte Carlo rollouts
REINFROCE使员工的是Monte Carlo计算returns，完整的算法如下：
REINFORCE 算法
Loop 
$\qquad 1.$使用policy $\pi\_{\theta}(a\_t|s\_t)$生成一个trajectory $\left[\tau^i \right]$
$\qquad$估计$\nabla\_{\theta}J(\theta) \approx \sum\_i (\sum\_t \nabla\_{\theta} \log\pi\_{\theta}(a\_t^i |s\_t^i )) (\sum\_t R(s\_t^i , a\_t^i ))$
$\qquad \theta\leftarrow \theta+\alpha\nabla\_{\theta}J(\theta)$
Until 收敛

### Policy Gradients Improvements
Policy gradient的方差很大，而且很难收敛，这是一大问题。
MC方法根据整个trajectory计算exact rewards，但是stochastic policy可能会在不同的episode采取不同的actions，一个小的改变可能会完全改变结果，MC方法没有bias但是有很大的方差。方差会影响深度学习的优化，一个采样的reward可能想要增加似然，另一个样本rewards可能想要减少似然，给出了冲突的梯度方向，影响收敛性。为了减少选择action造成的方差，我们需要减少样本rewards的方差：
$$\left( \sum\_{t=1}^T R(s\_{i,t}, a\_{i,t})\right)\tag{60}$$
增大PG中的batch size会减少方差。
但是增大batch size会降低sample efficiency。所以batch size不能增加太多，我们需要想其他的方法减少方差：

#### Causality
未来的action不应该改变过去的decisions：
$$\nabla\_{\theta}J(\theta) \approx \frac{1}{N} \sum\_{i=1}^N \sum\_{t=1}^T \nabla\_{\theta}\log\pi\_{\theta}(a\_{i,t}|s\_{i,t}) \left(\sum\_{t'=t}^T R(s\_{i,t'};a\_{i,t'})\right)\tag{61}$$
可以用$Q$代替$\sum_t R(s,a)$，
$$\nabla\_{\theta}J(\theta) \approx \frac{1}{N} \sum\_{i=1}^N \sum\_{t=1}^T \nabla\_{\theta}\log\pi\_{\theta}(a\_{i,t}|s\_{i,t}) Q\_{i,t}\tag{62}$$
为什么会减少方差？

#### Baseline
$$\nabla\_{\theta}J(\theta) \approx \frac{1}{N}\sum\_{i=1}^N \left(\sum\_{t=1}^T\nabla\_{\theta} \log\pi\_{\theta}(a\_{i,t}|s\_{i,t}\right) \left(\sum\_{t=1}^T R(s\_{i,t}, a\_{i,t})\right)\tag{63}$$
中$\sum\_{t=1}^T R(s\_{i,t}, a\_{i,t})$其实就是$Q(s,a)$，我们可以在上面减去一项，只要这一项和$\theta$无关就好，所以我们可以减去$V(s)$：
\begin{align\*}
\nabla\_{\theta} J(\theta) & \approx \frac{1}{N} \sum\_{i=1}^N \sum\_{t=1}^T \nabla\_{\theta}\log\pi\_{\theta}(a\_{i,t}|s\_{i,t})\left(Q(s\_{i,t}, a\_{i,t}) - V(s\_{i,t})\right)\\\\
& = \frac{1}{N} \sum\_{i=1}^N \sum\_{t=1}^T \nabla\_{\theta}\log\pi\_{\theta}(a\_{i,t}|s\_{i,t})\left(A(s\_{i,t}, a\_{i,t})\right)\\\\
\end{align\*}
直观上来首，RL感兴趣的是那些比平均值好的action。如果returns都是正的$(R(\tau)\ge 0)$，PG总是会提高这个trajectory发生的概率，即使它比其他的trajectory要低。考虑以下两个例子：
- Trajectory $A$的return是$10$，trajectory $B$的reward是$-10$
- Trajectory $A$的return是$10$，trajectory $B$的reward是$1$

在第一个例子中，PG会提高$A$发生的概率，降低$B$发生的概率。在第二个例子中，PG会提高$A$和$B$的概率。然而，对于我们来说，在两个例子中，我们都想要降低$B$发生的概率，提高$A$发生的概率。通过引入一个baseline，比如$V$，我们就可以实现这样的目的。


#### Vanilla Policy Gradient
给出一个使用baseline $b$的通用算法：
$$ \nabla\approx \hat{g} = \frac{1}{m} \sum\_{i=1}^m \nabla\_{\theta}\log P(\tau^{(i)} ;\theta)(R(\tau^{(i)} )-b)\tag{64}$$
Vanilla policy gradient算法
初始化policy 参数$\theta$，baselien $b$
for $i = 1, 2, \cdots$ do
$\qquad$使用当前policy $\pi\_{\theta}$收集trajectories
$\qquad$在每个trajectory的每一个timestep，计算
$\qquad\qquad$return $G\_t = \sum\_{t'=t}^{T-1} \gamma^{t'-t} R\_{t'}$
$\qquad\qquad$advantage的估计值$\hat{A}\_t = R\_t - b(s\_t)$
$\qquad$重新拟合baseline，最小化$\vert b(s\_t) - G\_t\vert^2 $
$\qquad$在所有trajectories和timesteps上求和估计$\hat{g}$
$\qquad$使用policy gradient estimate $\hat{g}$的估计$\hat{g}$更新policy 参数
end for

#### Reward discount
加上折扣因子：
$$Q^{\pi,\gamma}(s, a) \leftarrow r\_0 + \gamma r\_1 + \gamma^2 r\_2 + \cdots|s\_0 = s, a\_0 = a \tag{65}$$
得到：
$$\nabla\_{\theta}J(\theta) \approx \frac{1}{N} \sum\_{i=1}^N \left(\sum\_{t=1}^T \nabla\_{\theta}\log\pi\_{\theta}(a\_{i,t}|s\_{i,t}) \right) \left(\sum\_{t'=t}^T \gamma^{t'-t} R(s\_{i,t'};a\_{i,t'})\right)\tag{66}$$

## Policy Gradient with Approximation(使用近似的策略梯度)
因为$Q^{\pi} $是不知道的，我们希望用函数近似式子(8)中的$Q^{\pi} $，大致求出梯度的方向。用$f\_w:S\times A \rightarrow \mathbb{R}$表示$Q^{\pi} $的估计值。在策略$\pi$下，更新$w$的值:
$$\Delta w\_t\propto \frac{\partial}{\partial w}\left[\hat{Q}^{\pi} (s\_t,a\_t) - f\_w(s\_t,a\_t)\right]^2 \propto \left[\hat{Q}^{\pi} (s\_t,a\_t) - f\_w(s\_t,a\_t)\right]\frac{\partial f\_w(s\_t,a\_t)}{\partial w} \tag{67}$$
$\hat{Q}^{\pi} (s\_t,a\_t)$是$Q^{\pi} (s\_t,a\_t)$的一个无偏估计，当这样一个过程收敛到local optimum，$Q^{\pi} (s,a)$和$f\_w(s,a)$的均方误差最小时：
$$\epsilon(\omega, \pi) = \sum\_{s,a}\rho^{\pi} (s)\pi(a|s;\theta)(Q^{\pi} (s,a))^2 - f^{\pi} (s,a;\omega) \tag{68}$$
即导数等于$0$:
$$\sum\_s \rho^{\pi} (s)\sum\_a\pi(a|s;\theta)\left[Q^{\pi} (s,a) -f\_w (s,a;w)\right]\frac{\partial f\_w(s,a)}{\partial w}  = 0\tag{69}$$

### 定理2：Policy Gradient with Approximation Theorem
如果$f\_w$的参数$w$满足式子$20$，并且：
$$\frac{\partial f\_w(s,a)}{\partial w} = \frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)} = \frac{\partial \log \pi(s,a)}{\partial \mathbf{\theta}}\tag{70}$$
那么使用$f\_w(s,a)$计算的gradient和$Q^{\pi} (s,a)$计算的gradient是一样的：
$$\frac{\partial \rho}{\partial \theta} = \sum\_s\rho^{\pi} (s)\sum\_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}f\_w(s,a)\tag{71}$$

证明：
将式子$21$代入$20$得到：
\begin{align\*}
&\sum\_s\rho^{\pi} (s)\sum\_a\pi(s,a)\left[Q^{\pi} (s,a) -f\_w(s,a)\right]\frac{\partial f\_w(s,a)}{\partial w}\\\\
= &\sum\_s\rho^{\pi} (s)\sum\_a\pi(s,a)\left[Q^{\pi} (s,a) -f\_w(s,a)\right]\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\frac{1}{\pi(s,a)}\\\\
= &\sum\_s\rho^{\pi} (s)\sum\_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\left[Q^{\pi} (s,a) -f\_w(s,a)\right] \tag{72}\\\\
= & 0 \\\\
\end{align\*}
将式子$23$带入式子$8$：
\begin{align\*}
\frac{\partial \eta}{\partial \mathbf{\theta}} & = \sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a)\\\\
&= \sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a) - \sum\_s\rho^{\pi} (s)\sum\_a\frac{\partial \pi(s,a)}{\partial \mathbf{\theta}}\left[Q^{\pi} (s,a) -f\_w(s,a)\right]\\\\
&= \sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} \left[Q^{\pi} (s,a) - Q^{\pi} (s,a) +f\_w(s,a)\right]\\\\
&= \sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} f\_w(s,a) \tag{73}\\\\
\end{align\*}
得证$\sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}}Q^{\pi} (s,a) = \sum\_a \rho^{\pi} (s)\sum\_a\frac{\partial\pi(s,a)}{\partial\mathbf{\theta}} f\_w(s,a) $。

## Application to Deriving Algorithms and Advantages
给定一个参数化的policy，可以利用定理2推导出参数化value function的形式。比如，考虑在features上进行线性组合的Gibbs分布构成的policy：
$$\pi(a|s) = \frac{e\^{\theta^T \phi\_{sa} } }{\sum\_b e\^{\theta^T \phi\_{sb} }} , \forall s \in S, \forall a \in A \tag{74}$$
其中$\phi\_{s,a}$是state-action pair $s,a$的特征向量。满足式子$(21)$的公式如下：
$$\frac{\partial f\_w(s,a)}{\partial w} = \frac{\partial \pi(a|s)}{\partial \theta}\frac{1}{\pi(a|s)} = \phi\_{sa} - \sum\_b\pi(b|s)\phi\_{sb}\tag{75}$$
所以：
$$f\_w(s,a) = w^T \left[\phi\_{sa} - \sum\_b\pi(b|s)\phi\_{sb} \right]\tag{76}$$
也就是说，$f\_w$和policy $\pi$都是feature的线性组合，只不过每一个state处$f\_w$的均值都为$0$，$\sum\_a\pi(a|s)f\_w(s,a) = 0,\forall s\in S$。所以，其实我们可以认为$f\_w$是对advantage function $A^{\pi} (s,a) = Q^{\pi} (s,a)- V^{\pi} (s)$而不是$Q^{\pi} (s,a)$的一个近似。式子$(21)$中$f\_w$其实是一个相对值而不是一个绝对值。事实上，他们都可对以推广变成一个function加上一个value function。比如式子$(22)$可以变成$\frac{\partial\eta}{\partial \theta} = \sum\_s\rho^{\pi}(s) \sum\_a \frac{\partial \pi(a|s)}{\partial \theta}\left[f\_w(s,a) + v(s)\right]$，其中$v$是一个function，$v$的选择不影响理论结果，但是会影响近似梯度的方差。

## Convergence of Policy Iteration with Function Approximation(使用函数近似的策略迭代的收敛性)

### 定理3：Policy Iteration with Function Approximation
用$\pi$和$f\_w$表示policy和value function的可导函数，并且满足式子$(21)$。$\max\_{\theta,s,a,i,j} \vert\frac{\partial^2 \pi(a|s)}{\partial\theta\_i \partial\theta\_j} \vert\lt B\lt \infty$，假设$\left[\alpha\_k\right]\_{k=0}^{\infty}$是步长sequence，$\lim\_{k\rightarrow \infty}\alpha\_k = 0$，$\sum\_k \alpha\_k = \infty$。对于任何有界rewards的MDP来说，任意$\theta\_0$，$\pi\_k=\pi(\cdot, \theta\_k)$定义的$\left[\eta(\pi\_k)\right]\_{k=0}^{\infty}$，并且$w\_k = w$满足：
$$\sum\_s\rho^{\pi\_k} (s) \sum\_a\pi\_k(a|s)\left[Q^{\pi\_k} (s,a)-f\_w(s,a) \right]\frac{\partial f\_w(s,a)}{\partial w}=0 \tag{77}$$
$$\theta\_{k+1} = \theta\_k + \alpha\_k \sum\_s\rho^{\pi\_k}(s) \sum\_a\frac{\partial\pi\_k(s,a)}{\partial \theta}f\_{w\_k}(s,a) \tag{78}$$
一定收敛：$\lim\_{k\rightarrow \infty}\frac{\partial \rho(\pi\_k)}{\partial \theta} = 0$。


## 参考文献
1.https://papers.nips.cc/paper/1713-policy-gradient-methods-for-reinforcement-learning-with-function-approximation.pdf
2.https://papers.nips.cc/paper/2073-a-natural-policy-gradient.pdf
3.https://medium.com/@jonathan_hui/rl-policy-gradients-explained-9b13b688b146
4.https://medium.com/@jonathan_hui/rl-policy-gradients-explained-advanced-topic-20c2b81a9a8b
5.https://www.jianshu.com/p/af668c5d783d
