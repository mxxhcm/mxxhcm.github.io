---
title: reinforcement learning an introduction 第6章笔记
date: 2019-06-02 14:31:49
tags:
- 强化学习
categories: 强化学习
mathjax: true
---

## Temporal-Difference Learning
TD方法是DP和MC方法的结合，像MC一样，TD可以不需要model直接从experience中学习，像DP一样，TD是bootstrap的方法。
和之前一样，首先研究policy evaluation或者叫prediction问题，即给定一个policy $\pi$，估计$v\_{\pi}$。然后研究control问题，DP,TD,MC方法用的都是GPI方法，主要的不同点是prediction的解决方法。

## TD prediction
TD和MC都从采样得到的experience中求解prediction问题。给定一个policy $\pi$下的一个experience，TD和MC方法估计在这个experience中出现的non-terminal state $S_t$的$V$。MC方法等到整个experience的return知道以后，用这个return当做$V(S_t)$的target，一个every visit MC方法更新如下：
$$V(S_t) = V(S_t) + \alpha \left[G_t - V(S_t)]$$
其中$G_t$是从时刻$t$到这个episode结束的return，$\alpha$是一个常数的步长，这个方法叫做$constant-\alpha$ MC。MC方法必须等到一个episode结束，才能进行更新，因为只有这个时候$G_t$才知道。TD方法做了改进，它使用$t+1$时刻对应state的估计值作为target：
$$V(S_t) = V(S_t) + \alpha \left[R\_{t+1}+\gamma V(S\_{t+1}) - V(S_t)]$$
即只要有了到$S\_{t+1}$的transition并且接收到了reward $R\_{t+1}$就可以进行上述更新。
MC方法的target是$G_t$，而TD方法的target是$\gamma V(S\_{t+1}) + R\_{t+1})$，这种TD方法叫做$TD-0$或者$one step TD$，它是$TD(\lambda)$和$n-step TD$的一种特殊情况。下面是$TD(0)$的完整算法：
算法1 Tabular TD(0) for V
输入： 待评估的policy $\pi$
算法参数：步长$\alpha \in (0,1\]$
初始化： $V(s), \forall s\in S\^{+}，V(terminal) = 0$
**Loop** for each episode
$\qquad$初始化$S$
$\qquad$$A\leftarrow \pi(a|S)$
$\qquad$**Loop** for each step of episode
$\qquad\qquad$执行action A，得到$S'$和$R$
$\qquad\qquad$$$V(S) = V(S) + \alpha \left[R + \gamma V(S) - V(S)]$$
$\qquad\qquad$$S\leftarrow S'$
$\qquad$**until** S 是terminal state



