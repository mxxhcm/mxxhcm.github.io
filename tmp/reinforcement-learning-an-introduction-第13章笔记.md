---
title: reinforcement learning an introduction 第13章笔记.md
date: 2019-04-03 09:46:49
tags:
 - Policy Gradient
 - 强化学习
categories: 强化学习 
mathjax: true
---

## Policy gradient
这章介绍的是使用一个参数化策略(parameterized policy)直接给出action，而不用借助一个value funciton选择action。但是需要说一下的是，Policy gradient方法也可以学习一个Value function，但是value function是用来帮助学习policy parameters的，而不是用来选择action。我们用$\mathbf{\theta} \in R^{d'}$表示policy's parameters vector，用$\pi(a|s, \mathbf{\theta}) = Pr[A\_t = a|S\_t = s, \mathbf{\theta}\_t = \mathbf{\theta}]$表示environment在时刻$t$处于state $s$时，智能体根据参数为$\mathbf{\theta}$的策略$\pi$选择action $a$。
如果policy gradient方法使用了一个value function,它的权重用$\mathbf{w} \in R^d$表示，即$\hat{v}(s,\mathbf{w})$。

用$J(\mathbf{\theta})$表示policy parameters的标量performance measure。使用梯度上升(gradient ascent) 方法来最大化这个performance：
$$\mathbf{\theta}\_{t+1} = \mathbf{\theta}\_t + \alpha \widehat{\nabla J(\mathbf{\theta}\_t}),\tag{1}$$
其中$\widehat{\nabla J(\mathbf{\theta}\_t)} \in R^{d'}$是一个随机估计(stachastic estimate)，它的期望是performance measure对$\mathbf{\theta\_t}$的梯度。不管它们是否使用value function，这种方法就叫做policy gradient方法。既学习policy，又学习value function的方法被称为actor-critic，其中actor指的是学到的policy，critic指的是学习到的value funciton,通常是state value function。

## policy估计和它的优势
### 参数化policy的条件
policy可以用任何方式参数化，只要$\pi(a|s,\mathbf{\theta}),\mathbf{\theta}\in R^{d'}$对于它的参数$\mathbf{\theta}$是可导的，即只要$\nabla\_{\pi}(a|s,\mathbf{\theta})$（即：$\pi(a|s,\mathbf{\theta})$相对于$\mathbf{\theta}$的偏导数列向量）存在，并且$\forall s\in S, a\in A(s)$偏导数都是有限的即可。

### stochastic policy
为了保证exploration，通常策略是stochastic，而不是deterministic，即$\forall s,a,\mathbf{\theta}, \pi(a|s,\mathbf{\theta})\in (0,1)$

### 参数化方式的选择
#### softmax
对于有限且离散的action space，一个很自然的参数化方法就是对于每一个state-action对都计算一个参数化的数值偏好$h(s,a,\mathbf{\theta})\in R$。通过计算一个exponetial softmax，这个数值大的动作有更大的概率被选中：
$$\pi(a|s,\mathbf{\theta}) = \frac{e^{h(s,a,\mathbf{\theta} )}}{\sum_be^{h(s,b,\mathbf{\theta} )}}, \tag{2}$$
其中$b$是在state $s$下所有可能采取的动作，它们的概率加起来为$1$，这种方法叫做softmax in aciton preferences。

#### NN和线性方法
参数化还可以选择其他各种各样的方法，如AlphaGo中使用的NN，或者可以使用如下的线性方法：
$$h(s,a, \mathbf{\theta}) = \mathbf{\theta}^Tx(s,a), \tag{3}$$

### 优势
和action value方法相比，policy gradient有多个优势。
第一个优势是使用action preferences的softmax，同时用$\epsilon-greedy$算法用$\epsilon$的概率随机选择action得到的策略可以接近一个deterministic policy。
而单单使用action values的方法并不会使得策略接近一个deterministic policy，但是action-value方法会逐渐收敛于它的true values，翻译成概率来表示就是在$0$和$1$之间的一个概率值。但是action preferences方法不收敛于任何值，它们产生optimal stochastic policy，如果optimal policy是deterministic，那么optimal action的preferences应该比其他所有suboptimal actions都要高。

第二个优势是使用action preferences方法得到的参数化策略可以使用任意的概率选择action。在某些问题中，最好的approximate policy可能是stochastic的，actor-value方法不能找到一个stochastic optimal policy，它总是根据action value值选出来一个值最大的action，但是这时候的结果通常不是最优的。

第三个优势是policy parameterization可能比action value parameterization更容易学习。当然，也有时候可能是action value更容易。这个要根据情况而定

第四个优势是policy parameterizaiton比较容易添加先验知识到policy中。

## policy gradient理论
除了上节说的实用优势之外，还有理论优势。policy parameterization学到关于参数的一个连续函数，action probability概率可以平滑的变化。然而$\epsilon-greedy$算法中，action-value改变以后，action probability可能变化很大。很大程度上是因为policy gradient方法的收敛性要比action value方法强的多。因为policy的连续性依赖于参数，使得policy gradient方法接近于gradient ascent。
这里讨论episodic情况。定义perfromance measure是episode初始状态的值。假设每一个episode，都从state $s\_0$开始，定义：
$$J(\mathbf{\theta}) = v_{\pi_\mathbf{\theta}}(s_0), \tag{4}$$
其中$v\_{\pi\_\mathbf{\theta}}(s\_0)$是由参数$\mathbf{\theta}$确定的策略$\pi\_{\mathbf{\theta}}$的true value function。假设在episodic情况下，$\gamma=1$。

使用function approximation，一个需要解决的问题就是如何确保每次更新policy parameter，performance measure都有improvement。因为performence不仅仅依赖于action的选择，还取决于state的分布，然后它们都受policy parameter的影响。给定一个state，policy parameter对于actions，reward的影响，都可以相对直接的利用参数知识计算出来。但是policy parameter对于state 分布的影响是一个环境的函数，通常是不知道的。当梯度依赖于policy改变对于state分布的影响未知时，我们该如何估计performance相对于参数的梯度。

### Episodic case证明
为了简化表示，用$\pi$表示参数为$\theta$的policy，所有的梯度都是相对于$\mathbf{\theta}$求的
\begin{align\*}
\nabla v\_{\pi}(s) &= \nabla \[ \sum\_a \pi(a|s)q\_{\pi}(s,a)\], \forall s\in S \tag{5}\\\\
&= \sum\_a \[\nabla\pi(a|s)q\_{\pi}(s,a)\], \forall s\in S \tag{6}\\\\
&= \sum\_a\[\nabla\pi(a|s)q\_{\pi}(s,a) + \pi(a|s)\nabla q\_{\pi}(s,a)\] \tag{7}\\\\
&= \sum\_a\[\nabla\pi(a|s)q\_{\pi}(s,a) + \pi(a|s)\nabla \sum\_{s',r}p(s',r|s,a)(r+\gamma v\_{\pi}(s'))\] \tag{8}\\\\
&= \sum\_a\[\nabla\pi(a|s)q\_{\pi}(s,a) + \pi(a|s) \nabla \sum\_{s',r}p(s',r|s,a)r + \pi(a|s)\nabla \sum\_{s',r}p(s',r|s,a)\gamma v\_{\pi}(s'))\] \tag{9}\\\\
&= \sum\_a\[\nabla\pi(a|s)q\_{\pi}(s,a) + 0 + \pi(a|s)\sum\_{s'}\gamma p(s'|s,a)\nabla v\_{\pi}(s') \] \tag{10}\\\\
&= \sum\_a\[\nabla\pi(a|s)q\_{\pi}(s,a) + 0 + \pi(a|s)\sum\_{s'}\gamma p(s'|s,a)\\\\
&\ \ \ \ \ \ \ \ \sum\_{a'}[\nabla\pi(a'|s')q\_{\pi}(s',a') + \pi(a'|s')\sum\_{s''}\gamma p(s''|s',a')\nabla v\_{\pi}(s''))] \],  \tag{11}展开\\\\
&= \sum\_{x\in S}\sum\_{k=0}^{\infty}Pr(s\rightarrow x, k,\pi)\sum\_a\nabla\pi(a|x)q\_{\pi}(x,a) \tag{12}
\end{align\*}
第(5)式使用了$v\_{\pi}(s) = \sum\_a\pi(a|s)q(s,a)$进行展开。第(6)式将梯度符号放进求和里面。第(7)步使用product rule对q(s,a)求导。第(8)步利用$q\_{\pi}(s, a) =\sum\_{s',r}p(s',r|s,a)(r+v\_{\pi}(s')$ 对$q\_{\pi}(s,a)$进行展开。第(9)步将(8)式进行分解。第(10)步对式(9)进行计算，因为$\sum\_{s',r}p(s',r|s,a)r$是一个定制，求偏导之后为$0$。第(11)步对生成的$v\_{\pi}(s')$重复(5)-(10)步骤，得到式子(11)。如果对式子(11)中的$v\_{\pi}(s)$一直展开，就得到了式子(12)。式子(12)中的$Pr(s\rightarrow x, k, \pi)$是在策略$\pi$下从state $s$经过$k$步转换到state $x$的概率，这里我有一个问题，就是为什么，$k$可以取到$\infty$，后来想了想，因为对第(11)步进行展开以后，可能会有重复的state，重复的意思就是从状态$s$开始，可能会多次到达某一个状态$x$，$k$就能取很多次，大不了$k=\infty$的概率为$0$就是了。

所以，对于$v\_{\pi}(s\_0)$，就有：
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \nabla\_{v\_{\pi}}(s\_0)\\\\
&= \sum\_{s\in S}\( \sum\_{k=0}^{\infty}Pr(s\_0\rightarrow s,k,\pi) \) \sum\_a\nabla\_{\pi}(a|s)q\_{\pi}(s,a)\\\\
&=\sum\_{s\in S}\eta(s)\sum\_a \nabla\_{\pi}(a|s)q\_{\pi}(s,a)\\\\
&=\sum\_{s'\in S}\eta(s')\sum\_s\frac{\eta(s)}{\sum\_{s'}\eta(s')}\sum\_a \nabla\_{\pi}(a|s)q\_{\pi}(s,a)\\\\
&=\sum\_{s'\in S}\eta(s')\sum\_s\mu(s)\sum\_a \nabla\_{\pi}(a|s)q\_{\pi}(s,a)\\\\
&\propto \sum\_{s\in S}\mu(s)\sum\_a\nabla\pi(a|s)q\_{\pi}(s,a)
\end{align\*}
最后，我们可以看出来performance对policy求导不涉及state distribution的导数。Episodic 情况下的策略梯度如下所示：
$$\nabla J(\mathbf{\theta})\propto \sum_{s\in S}\mu(s)\sum_aq_{\pi}(s,a)\nabla\pi(a|s,\mathbf{\theta}), \tag{13}$$
其中梯度是performacne指标$J$关于$\mathbf{\theta}$的偏导数列向量，$\pi$是参数$\mathbf{\theta}$对应的策略。在episodic情况下，比例常数是一个episode的平均长度，在continuing情况下，常数是$1$，实际上这个正比于就是一个等式。分布$\mu$是策略$\pi$下的on-policy分布。

## REINFORCE: Monte Carlo Policy Gradient
对于式子(1)，我们需要进行采样，让样本梯度的期望正比于performance measure对于$\mathbf{\theta}$的真实梯度。比例系数不需要确定，因为步长$\alpha$的大小是手动设置的。Policy gradient理论给出了一个正比于gradient的精确表达式，我们要做的就是选择采样方式，它的期望等于或者接近policy gradient理论给出的值。

### all-actions
使用随机变量的期望替换对随机变量求和的取值，我们可以将式子(13)进行如下变化：
\begin{align\*}
\nabla J(\mathbf{\theta})&\propto \sum\_{s\in S}\mu(s)\nabla\pi(a|s,\mathbf{\theta})\sum\_aq\_{\pi}(s,a)\\\\
&=\mathbb{E}\_{\pi}\left[\nabla\pi(a|S\_t,\mathbf{\theta})\sum\_aq\_{\pi}(S\_t,a)\right]\tag{14}
\end{align\*}
接下来，我们可以实例化该方法：
$$\mathbf{\theta}\_{t+1} = \mathbf{\theta}\_t+\alpha\sum\_a\hat{q}(S\_t,s,\mathbf{w})\nabla\pi(a|S\_t,\mathbf{\theta}), \tag{15}$$
其中$\hat{q}$是$q\_{\pi}$的估计值，这个算法被称为all-actions方法，因为它的更新涉及到了所有的action。然而，我们这里介绍的REINFORCE仅仅使用了$t$时刻的action $A\_t$。。

### REINFORCE
和引入$S\_t$的方法一样，使用随机变量的期望代替对与随机变量的可能取值进行求和，我们在式子(14)中引入$A\_t$，
\begin{align\*}
\nabla J(\mathbf{\theta}) &= \mathbb{E}\_{\pi}\left[\sum\_aq\_{\pi}(S\_t,a)\nabla\pi(a|S\_t,\mathbf{\theta})\right]\\\\
& = \mathbb{E}\_{\pi}\left[\sum\_aq\_{\pi}(S\_t,a)\pi(a|S\_t,\mathbf{\theta})\frac{\nabla\pi(a|S\_t,\mathbf{\theta})}{\pi(a|S\_t,\mathbf{\theta})}\right]\\\\
& = \mathbb{E}\_{\pi}\left[q\_{\pi}(S\_t,A\_t)\frac{\nabla\pi(A\_t|S\_t,\mathbf{\theta})}{\pi(A\_t|S\_t,\mathbf{\theta})}\right]\\\\
\end{align\*}
