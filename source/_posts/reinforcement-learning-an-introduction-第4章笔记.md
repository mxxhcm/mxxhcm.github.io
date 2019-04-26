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
&= \sum_{s',r} p(s',r|s,a) \left[r + \gamma max_a q_{\*}(s',a')\right] \tag{2}
\end{align\*}

