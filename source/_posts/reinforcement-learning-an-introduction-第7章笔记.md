---
title: reinforcement learning an introduction 第7章笔记
date: 2019-07-30 09:59:50
tags:
 - n-steps
 - 强化学习
categories: 强化学习`
mathjax: true
---


## n-step Bootstrapping
这章要介绍的n-step TD方法，将MC和one-step TD用一个统一的框架整合了起来。在一个任务中，两种方法可以无缝切换。n-step方法生成了一系列方法，MC在一头，one-step在另一头。最好的方法往往不是MC也不是one-step TD。
one-step每隔一个timestep进行bootstrapping，在许多需要及时更新变化的问题，这种方法很有用，但是经历了长时间的稳定变化之后，bootstrap的效果会更好。n-step就是将one-step TD方法中time interval的one改成了n。
n-step方法的idea和eligibility traces很像，eligibility traces可以同时使用多个不同的time intervals进行bootstarp。

## n-step TD Prediction
使用policy $\pi$生成sample episodes，然后估计$v_{\pi}$，MC方法使用episode中每一个state的return进行更新，使用的是无穷步reward之和。TD方法使用每一个state执行完某一个action之后的下一个reward加上下一个state的估计值进行更新。n-step使用的是每一个state接下来n步的reward之和加上n步后的state的估计值。相应的backup diagram如下所示：
![n_step_diagram](n_step_backup_diagram.png)

n-step方法还是属于TD方法，因为n-step的更新还是基于时间维度上不同estimate的估计进行的。
> n-step updates are still TD methods because they still chagne an erlier estimate based on how it differs from a later estimate. Now the later estimate is not one step later, but n steps later。

正式的，假设$S_t$的estimated value是基于state-reward sequence,$S_t, R_{t+1}, S_{t+1}, R_{t+2}, \cdots, R_T,S_T$得到的。
MC方法更新基于completer return：
$$G_T = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots + \gamma^{T-t-1}R_T$$
one-step TD更新基于one-step return：
$$G_{t:t+1} = R_{t+1}+ \gamma V_t(S_{t+1}) $$
two-step TD更新基于two-step return：
$$G_{t:t+2} = R_{t+1}+ \gamma V_t(S_{t+1}) + \gamma^2 V_{t+1}(S_{t+2}) $$
n-step TD更新n-step return：
$$G_{t:t+n} = R_{t+1}+ \gamma V_t(S_{t+1}) + \gamma^2 V_{t+1}(S_{t+2}) +\cdots + \gamma^{n-1} R_{t+n} + \gamma^n V_{t+n-1}(S_{t+n}), n\ge1, 0\le t\le T-n $$
所有的n-step方法都可以看成对full return 的近似，计算n steps的rewards之和以后，使用$V_{t+n-1}(S_{t+N})$近似其余的项。如果$t+n \ge T$时，所有的超过时间步的reward以及estimated value当做$0$，相当于定义$G_{t:t+1} = G_t, if\quad t+n \ge T$

当$n>1$时，不能使用real time的算法，因为只有在$t+n$时刻，$R_{t+n}$才是已知的，$S_{t+n}$也才是已知的，这个时候才能计算使用$t+n-1$时刻的$V$近似估计$V_{t+n-1}(S_{t+n})$。将n-step return当做n-step TD的target，得到的更新公式如下：
$$V_{t+n}(S_t) = V_{t+n-1} (S_t) + \gamma(G_{t:t+n} - V_{t+n-1}(S_{t}))$$
当更新$V_{t+n}(S_t)$时，所有其他states不变，$V_{t+n}(s) = V_{t+n-1}(s), \forall s\neq S_t$。在每个episode的前$n-1$步中，不进行任何更新。所以为了弥补这几步，在每个episode结束（在terminal之后以及下一个episode开始之前）的时候加一些额外的updates。完整的n-step TD算法如下：
n-step TD估计$V\approx v_{\pi}$
输入：一个policy $\pi$
算法参数：step size $\alpha \in (0, 1]$，正整数$n$
随机初始化$V(s), \forall s\in S$
Loop for each episode
$\qquad$初始化 $S_0 \neq terminal$
$\qquad$$T\leftarrow \infty$
$\qquad$Loop for $t=0, 1, 2, \cdots:$
$\qquad$ If $t\lt T$, then
$\qquad$ 根据$\pi(\cdot|S_t)$执行action
$\qquad$ 接收并记录$R_{t+1}, S_{t+1}$
$\qquad$ 如果$S_{t+1}$是terminal ，更新$T\leftarrow t+1$
$\qquad$ End if
$\qquad$ $\tau \leftarrow t - n + 1, \tau$是当前更新的时间
$\qquad$ If $\tau \ge 0$, then
$\qquad$ $G\leftarrow \sum_{i=\tau+1}^{min(\tau+n, T)} \gamma^{i-\tau -1} R_i$
$\qquad$ 如果$\tau+n \lt T$，那么$G\leftarrow G+ \gamma^n V(S_{\tau+n})$
$\qquad$ $V(S_{\tau}) \leftarrow V(S_{\tau}) +\alpha [G-V(S_{\tau})]$
$\qquad$ End if 
Until $\tau = T-1$

## n-step Sarsa
## n-step Off-policy Learning
## 
## 参考文献
1.《reinforcement learning an introduction》第二版
2. 
