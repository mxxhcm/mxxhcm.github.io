---
title: reinforcement learning an introduction 第4章笔记
date: 2019-04-07 23:46:17
tags:
 - 动态规划
 - DP
 - 强化学习
categories: 强化学习
mathjax: true
---

## Dynamic Programming
DP指的是给定环境的模型(MDP)，用来计算智能体最优策略的一系列算法。经典的DP算法应用场景很有限，因为它需要知道环境的模型，并且需要很高的计算代价，但是这个思路很重要，是其他方法的基础。其他的许多算法都是在用更少的代价和关于环境更少的信息获得和DP算法尽可能接近的performance。
通常我们假定环境是一个finite MDP，也就是state, action, reward是finite。尽管DP可以应用于continuous的state和action space，但是只能应用在几个特殊场景上。一个常用的做法是将continuous state和action quantize(量化)，然后使用finite MDP。
DP的关键点在于使用value function寻找好的policy。使用第三章定义的value function，在找到了optimal value function之后，我们可以使用Bellman optimal equation找到optimal policy，参见[第三章推导](https://mxxhcm.github.io/2018/12/21/reinforcement-learning-an-introduction-%E7%AC%AC3%E7%AB%A0%E7%AC%94%E8%AE%B0/)：

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
只要$\gamma \lt 1$或者存在terminal state，那么$v_{\pi}$的必然存在而且是唯一的。这个我觉得是迭代法解方程的条件。数值分析上有证明。
如果环境的变换$p$是已知的，我们可以列出方程组，直接求解出每个状态$s$的$v(s)$。这里采用迭代法求解，随机初始化$v_0$，使用式子$\tag{4}$进行更新：
\begin{align\*}
v_{k+1}(s) &= \mathbb{E}\left[R_{t+1} + \gamma v_k(S_{t+1}) \S_t=s\right]\\
&= \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_k(s') \right] \tag{5}
\end{align\*}
$v_k=v_{\pi}$是一个fixed point，因为Bellman equation确保了这个条件。当$k\rightarrow \infty$时收敛到$v_{\pi}$。这个算法叫做iterative policy evaluation。
在每一次$v_k$到$v_{k+1}$的迭代过程中，所有的$v(s)$都会被更新，$s$的旧值被后继状态$s'$的旧值加上reward替换，正如公式$(5)$中体现的那样。这个目标值被称为expected update，因为它是基于所有$s'$的期望计算出来的（有环境的模型），而不是通过对$s'$采样计算的。
在实现iterative policy evaluation的时候，每一次迭代，都需要重新计算所有$s$的值。这里有一个问题，就是你在每次更新$s$的时候，使用的$s'$如果在本次迭代过程中已经被更新过了，那么是使用更新过的$s'$，还是使用没有更新的$s'$，这就和迭代法中的雅克比迭代以及高斯赛德尔迭代很像，如果使用更新后的$s'$，这里我们叫它in-place的算法，否则就不是。具体那种方法收敛的快，还是要看应用场景的，并不是in-place的就一定收敛的快，这是在数值分析上学到的。
下面给出in-place版本的iterative policy evation算法伪代码。
输入需要evaluation的policy $\pi$
给出算法的参数：阈值$\theta\gt 0$，当两次更新的差值小于这个阈值的时候，就停止迭代，随机初始化$V(s),\forall s\in S^{+}$，除了$V(terminal) = 0$。
**Loop**
$\qquad \delta \leftarrow 0$
$\qquad$ **for** each $s\in S$
$\qquad\qquad v\leftarrow V(s)$ （保存迭代之前的$V(s)$）
$\qquad\qquad V(s)\leftarrow\sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_k(s') \right] $
$\qquad\qquad \nabla \leftarrow max(\delta,|v-V(s)|)$
$\qquad$**end for** 
**until** $\delta \lt \theta$
