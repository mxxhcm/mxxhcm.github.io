---
title: reinforcement learning an introduction 第5章笔记
date: 2019-04-29 15:53:02
tags:
- 强化学习
- 蒙特卡洛
categories: 强化学习
---


## Monte Carlo Methods
这一章介绍的是Monte Carlo方法，和DP不一样的是，它不需要直到环境的信息，只需要experience即可--从真实交互或者仿真环境中得到的state,action,reward采样序列。从真实交互中学习不需要直到环境的信息，从仿真环境中学习需要一个model，但是这个model只用于生成sample transition，并不需要像DP那样需要所有transition的完整概率分布。在很多情况下，生成experience sample要比显示的得到概率分布容易很多。

蒙特卡洛算法基于average sample returns估计值函数。为了保证returns是可用的，这里定义蒙特卡洛算法是episodic的，即所有的experience都有一个terminal state。只有在一个episode结束的时候，value estimate和policy才会改变。蒙塔卡洛算法可以在episode和episode实现增量式，不能在step和step之间实现增量式。(Monte Carlo methods can thus be incremental in an episode-by-episode sense, but not in a step-by-step online sense.)
蒙特卡洛算法通过从采样和average returns对每一个state-action进行评估。在一个state采取action得到的return取决于同一个episode后续状态的action，因为所有的action都是在不断学习中采取，从早期state的角度来看，这个问题是non-stationary的。为了解决non-stationary问题，采用GPI中的idea。DP从已知的MDP中计算value function，蒙特卡洛使用MDP的sample returns学习value function。然后value function和对应的policy交互获得好的value和policy。
这一章就是把DP中的各种想法推广到了MC上，比如prediction和control问题，DP使用的是整个MDP，而MC使用的是MDP的采样。


## Monte Carlo Prediction
预测问题就是估计value function，从state value function说起。最简单的想法就是使用experience估计value function，通过对每个state experience中return做个average。
这里主要介绍两个算法，一个叫做$first visit MC method$，另一个是$every visit MC method$。比如要估计策略$\pi$下的$v(s)$，通过采样一系列经过$s$的episodes，$s$在每一个episode中出现一次叫做一个$visit$，一个$s$可能在一个episode中出现多次。$first visit$就是只取第一次$visit$估计$v(s)$，$every visit$就是每一次$visit$都用。
下面给出$first visit$的算法：
**First visit MC preidction**
**输入** 被评估的policy $\pi$
**初始化**:
$\qquad V(s)\in R,\forall s \in S$
$\qquad Returns(s)=[],\forall s \in S$
**Loop** for each episeode:
$\qquad$生成一个episode
$\qquadG\leftarrow 0$
$\qquad$**Loop** for each step, $t= T-1,T-2, \cdots, 1$
$\qquad\qquad G\leftarrow G + \gamma R_t$
$\qquad\qquad$ 如果$S_t$没有在$S_0, \cdots , S_{t-1}$中出现过
$\qquad\qquad\qquad Returns(S_t).apppend(G)$
$\qquad\qquad\qquad V(S_t)\leftarrow = average(Returns(S_t))$ 
$every visit$的话，没有加上判断$S_t$是否出现过。

## Monte Carlo Estimation of Action Values

## Monte Carlo Control

## Mnote Carlo Control without Exploring Starts

## Off-policy Prediction via Importance Sampling

## Incremental Implementation

## Off-policy Monte Carlo Control

## Discounting-aware Importance Sampling

## Summary
 
