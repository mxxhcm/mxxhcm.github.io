---
title: reinforcement learning an introduction 第4章笔记
date: 2019-04-07 23:46:17
tags:
 - 动态规划
 - 强化学习
categories: 强化学习
mathjax: true
---

## Dynamic Programming
DP指的是给定环境的模型，通常是一个MDP，计算智能体最优策略的一类算法。经典的DP算法应用场景有限，因为它需要环境的模型，以及很高的计算代价，但是DP的思路是很重要的。其他的许多算法都是在减少计算代价和环境信息的前提下尽可能获得和DP接近的性能。
通常我们假定环境是一个有限(finite)的MDP，也就是state, action, reward都是有限的。尽管DP可以应用于连续(continuous)的state和action space，但是只能应用在几个特殊的场景上。一个常见的做法是将连续state和action quantize(量化)，然后使用有限MDP。
DP关键在于使用value function寻找好的policy，在找到了满足Bellman optimal equation的optimal value function之后，可以找到optimal policy，参见[第三章推导](https://mxxhcm.github.io/2018/12/21/reinforcement-learning-an-introduction-%E7%AC%AC3%E7%AB%A0%E7%AC%94%E8%AE%B0/)：
Bellman optimal equation:
\begin{align\*}
v_{\*}(s) &= max_a\mathbb{E}\left[R_{t+1}+\gamma v_{\*}(S_{t+1})|S_t=s,A_t=a\right] \\
&= max_a \sum_{s',r} p(s',r|s,a){\*}\left[r+\gamma v_{\*}(s')\right]  \tag{1}
\end{align\*}

\begin{align\*}
q_{\*}(s,a) &= \mathbb{E}\left[R_{t+1}+\gamma max_{a'}q_{\*}(S_{t+1},a')|S_t=s,A_t = a\right]\\
&= \sum_{s',r} p(s',r|s,a) \left[r + \gamma max_a q\_{\*}(s',a')\right] \tag{2}
\end{align\*}

## Policy Evaluation(Prediction)
给定一个policy，计算state value function的过程叫做policy evaluation或者prediction problem。
根据$v(s)$和它的后继状态$v(s')$之间的关系：
\begin{align\*}
v_{\pi}(s) &= \mathbb{E}_{\pi}[G_t|S_t = s]\\
&= \mathbb{E}_{\pi}\left[R_{t+1}+\gamma G_{t+1}|S_t = s\right]\\
&= \sum_a \pi(a|s)\sum_{s'}\sum_rp(s',r|s,a) \left[r + \gamma \mathbb{E}_{\pi}\left[G_{t+1}|S_{t+1}=s'\right]\right] \tag{3}\\
&= \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_{\pi}(s') \right] \tag{4}\\
\end{align\*}
只要$\gamma \lt 1$或者存在terminal state，那么$v_{\pi}$的必然存在且唯一。这个我觉得是迭代法解方程的条件。数值分析上有证明。
如果环境的转换概率$p$是已知的，可以列出方程组，直接求解出每个状态$s$的$v(s)$。这里采用迭代法求解，随机初始化$v_0$，使用式子$(4)$进行更新：
\begin{align\*}
v_{k+1}(s) &= \mathbb{E}\left[R_{t+1} + \gamma v_k(S_{t+1})\ S_t=s\right]\\
&= \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_k(s') \right] \tag{5}
\end{align\*}
直到$v_k=v_{\pi}$到达一个fixed point，Bellman equation满足这个条件。当$k\rightarrow \infty$时收敛到$v_{\pi}$。这个算法叫做iterative policy evaluation。
在每一次$v_k$到$v_{k+1}$的迭代过程中，所有的$v(s)$都会被更新，$s$的旧值被后继状态$s'$的旧值加上reward替换，正如公式$(5)$中体现的那样。这个目标值被称为expected update，因为它是基于所有$s'$的期望计算出来的（利用环境的模型），而不是通过对$s'$采样计算的。
在实现iterative policy evaluation的时候，每一次迭代，都需要重新计算所有$s$的值。这里有一个问题，就是你在每次更新$s$的时候，使用的$s'$如果在本次迭代过程中已经被更新过了，那么是使用更新过的$s'$，还是使用没有更新的$s'$，这就和迭代法中的雅克比迭代以及高斯赛德尔迭代很像，如果使用更新后的$s'$，这里我们叫它in-place的算法，否则就不是。具体那种方法收敛的快，还是要看应用场景的，并不是in-place的就一定收敛的快，这是在数值分析上学到的。
下面给出in-place版本的iterative policy evation算法伪代码。
**iterative policy evation 算法**
**输入**需要evaluation的policy $\pi$
给出算法的参数：阈值$\theta\gt 0$，当两次更新的差值小于这个阈值的时候，就停止迭代，随机初始化$V(s),\forall s\in S^{+}$，除了$V(terminal) = 0$。
**Loop**
$\qquad \delta \leftarrow 0$
$\qquad$ **for** each $s\in S$
$\qquad\qquad v\leftarrow V(s)$ （保存迭代之前的$V(s)$）
$\qquad\qquad V(s)\leftarrow\sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_k(s') \right] $
$\qquad\qquad \nabla \leftarrow max(\delta,|v-V(s)|)$
$\qquad$**end for** 
**until** $\delta \lt \theta$

## Policy Improvement
为什么要进行policy evaluation，或者说为什么要计算value function？
其中一个原因是为了找到更好的policy。假设我们已经知道了一个deterministic的策略$\pi$，但是在其中一些状态，我们想要知道是不是有更好的action选择，如$a\neq \pi(s)$的时候，是不是这个改变后的策略会更好。好该怎么取评价，这个时候就可以使用值函数进行评价了，在某个状态，我们选择$a \neq \pi(s)$，在其余状态，依然遵循策略$\pi$。用公式表示为：
\begin{align\*}
q_{\pi}(s,a) &= \mathbb{E}\left[R_{t+1}+\gamma v_{\pi}(S_{t+1})|S_t=s,A_t = a\right]\\
&=\sum_{s',r}p(s',r|s,a)\left[r+\gamma v_{\pi}(s')\right] \tag{6}
\end{align\*}
那么，这个值是是比$v(s)$要大还是要小呢？如果比$v(s)$要大，那么这个新的策略就比$\pi$要好。
用$\pi$和$\pi'$表示任意一对满足下式的policy：
$$q_{\pi}(s,\pi'(s)) \ge v_{\pi}(s) \tag{7}$$
那么$\pi'$至少和$\pi$一样好。可以证明，任意满足$(7)$的$s$都满足下式：
$$v_{\pi'}(s) \ge v_{\pi}(s) \tag{8}$$
对于我们提到的$\pi$和$\pi'$来说，除了在状态$s$处，$v_{\pi'}(s) = a \neq v_{\pi}(s)$，在其他状态处$\pi$和$\pi'$是一样的，都有$q_{\pi}(s,\pi'(s)) = v_{\pi}(s)$。而在状态$s$处，如果$q_{\pi}(s,a) \gt v_{\pi}(s)$，注意这里$a=\pi'(s)$，那么$\pi'$一定比$\pi$好。
证明：
\begin{align\*}
v_{\pi}(s) &\le q_{\pi}(s,\pi'(s))\\
& = \mathbb{E}\left[R_{t+1} + \gamma v_{\pi}(S_{t+1})|S_t = s, A_t = \pi'(s) \right]\\
& = \mathbb{E}_{\pi'}\left[R_{t+1} + \gamma v_{\pi}(S_{t+1})|S_t = s \right]\\
& \le \mathbb{E}_{\pi'}\left[R_{t+1} + \gamma q_{\pi}(S_{t+1},\pi'(S_{t+1}))|S_t = s \right]\\
& = \mathbb{E}_{\pi'}\left[R_{t+1} + \gamma \mathbb{E}_{\pi'}\left[R_{t+2} + v_{\pi}(S_{t+2})|S_{t+1}=, A_{t+1}=\pi'(S_{t+1})|S_t = s \right\right]\\
& = \mathbb{E}_{\pi'}\left[ \right]\\
& = \mathbb{E}_{\pi'}\left[ \right]\\
& = \mathbb{E}_{\pi'}\left[ \right]\\
\end{align\*}
