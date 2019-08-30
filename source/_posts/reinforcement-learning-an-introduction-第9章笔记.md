---
title: reinforcement learning an introduction 第9章笔记
date: 2019-04-04 10:14:08
tags:
 - 函数近似 
 - on-policy
 - 强化学习
 - 值函数
categories: 强化学习
mathjax: true
---

## On-policy Prediction with Approximation
这一章讲的是利用on-policy的数据估计函数形式的值函数，on-policy就是说利用一个已知的policy $\pi$生成的experience来估计$v_{\pi}$。和之前讲的不同的是，前面几章讲的是表格形式的值函数，而这一章是使用参数为$\mathbf{w}\in R^d$的函数表示。即$\hat{v}(s,\mathbf{w})\approx v_{\pi}(s)$表示给定一个权值vector $\mathbf{w}$，state $s$的状态值。这个函数可以是任何形式的，可以是线性函数，也可以是神经网络，还可以是决策树。

## 值函数估计
目前这本书介绍的所有prediction方法都是更新某一个state的估计值函数向backed-up value（或者叫update target）值移动。我们用符号$s\mapsto u$表示一次更新。其中$s$是要更新的状态，$u$是$s$的估计值函数的update target。例如，Monte Carlo更新的value prediction是：$S_t \mapsto G_t$，TD(0)的update是：$S_t \mapsto R_{t+1} + \gamma \hat{v}(S_{t+1}, \mathbf{w}\_t)$，$n$-step TD update是：$S_t \mapsto G_{t:t+n}$。在DP policy evaluation update中是：$s\mapsto E_{\pi}[R_{t+1}+\gamma\hat{v}(S_{t+1}, \mathbf{w}_t)| S_t =s]$，任意一个状态$s$被更新了，同时在其他真实experience中遇到的$S_t$也被更新了。

之前表格的更新太trivial了，更次更新$s$向$u$移动，其他状态的值都保持不变。现在使用函数实现更新，在状态$s$处的更新，可以一次性更新很多个其他状态的值。就像监督学习学习input和output之间的映射一样，我们可以把$s\mapsto g$的更新看做一个训练样本。这样就可以使用很多监督学习的方法学习这样一个函数。
但是并不是所有的方法都适用于强化学习，因为许多复杂的神经网络和统计学方法都假设训练集是静态不变的。然而强化学习中，学习是online的，即智能体不断地与环境进行交互产生新的数据，这就需要这个方法能够从不断增加的数据中高效的学习。
此外，强化学习通常需要function approximation能够处理target function不稳定的情况，即target function随着事件在不断的变化。比如，在基于GPI的control方法中，在$\pi$不断变化的情况下，我们想要学习出$q_{\pi}$。即使policy保持不变，如果使用booststrapping方法（DP和TD学习），训练样本的target value也在不断的改变，因为下一个state的value值在不断的改变。所以不能处理这些不稳定情况的方法有点不适合强化学习。

## 预测目标(The Prediction Objective)
表格形式的值函数最终都会收敛到真值，状态值之间也都是解耦的，即更新一个state不影响另一个state。
但是使用函数拟合，更新一个state的估计值就会影响很多个其他状态，并且不可能精确的估计所有states的值。假设我们的states比weights多的多，让一个state的估计更精确也意味着使得其他的state越不accurate。我们用一个state $s$上的分布,$\mu(s)\ge 0,\sum_s\mu(s)=1$代表对每个state上error的权重。然后使用$\mu(s)$对approximate value $\hat{v}(s,\mathbf{w})$和true value $v_{\pi}(s)$的squared error进行加权，得到Mean Squared Value Error，表示为$\bar{VE}$：
$$\bar{VE}(\mathbf{w}) = \sum_{s\in S}\mu(s)[v_{\pi}(s) - \hat{v}(s, \mathbf{w})]^2$$
通常情况下，$\mu(s)$是在state $s$处花费时间的百分比。在on-policy训练中，这叫做on-policy分布。在continuing tasks中，策略$\pi$下的on-policy分布是一个stationary distribution。
在episodic tasks中，on-policy分布有一些不同，因为它还取决于每个episodic的初始状态，用$h(s)$表示在一个episodic开始状态为$s$的概率，用$\eta(s)$表示在一个回合中，state $s$平均被访问的次数。
$$\eta(s) = h(s) + \sum_{\bar{s}}\eta(\bar{s})\sum_a\pi(a|\bar{s})p(s|\bar{s},a), forall\ s \in S$$
其中$\bar{s}$是$s$的前一个状态，$s$处的时间为以状态$s$开始的概率$h(s)$加上它由前一个状态$\bar{s}$转换过来消耗的时间。
列出一个方程组，可以解出来$\eta(s)$的期望值。然后进行归一化，得到：
$$\mu(s)=\frac{\eta{s}}{\sum_{s'}\eta{s'}}, \forall s \in S.$$
这是没有折扣因子的式子，如果有折扣因子的话，可以看成一种形式的
