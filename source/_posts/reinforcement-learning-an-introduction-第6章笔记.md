---
title: reinforcement learning an introduction 第6章笔记
date: 2019-06-02 14:31:49
tags:
- 强化学习
categories: 强化学习
mathjax: true
---

## 简介
TD方法是DP和MC方法的结合，像MC一样，TD可以不需要model直接从experience中学习，像DP一样，TD是bootstrap的方法。

## Temporal-Difference Learning
本章的结构和之前一样，首先研究policy evaluation或者叫prediction问题，即给定一个policy $\pi$，估计$v\_{\pi}$。然后研究control问题，DP,TD,MC方法用的都是GPI方法，不同点在于prediction问题的解决方法。

## TD prediction
TD和MC都从采样得到的experience中求解prediction问题。给定一个policy $\pi$下的一个experience，TD和MC方法估计在这个experience中出现的non-terminal state $S_t$的$V$。MC方法等到整个experience的return知道以后，用这个return当做$V(S_t)$的target，一个every visit MC方法更新如下：
$$V(S_t) = V(S_t) + \alpha \left[G_t - V(S_t)\right]\tag{1}$$
其中$G_t$是从时刻$t$到这个episode结束的return，$\alpha$是一个常数的步长，这个方法叫做$constant-\alpha$ MC。MC方法必须等到一个episode结束，才能进行更新，因为只有这个时候$G_t$才知道。TD方法做了改进，它使用$t+1$时刻对应的state $V(S\_{t+1})$的估计值和reward $R\_{t+1}$作为target：
$$V(S_t) = V(S_t) + \alpha \left[R\_{t+1}+\gamma V(S\_{t+1}) - V(S_t)\right]\tag{2}$$
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
$\qquad\qquad V(S) = V(S) + \alpha \left[R + \gamma V(S') - V(S)\right]$
$\qquad\qquad$$S\leftarrow S'$
$\qquad$**until** S 是terminal state

$TD(0)$是bootstrap方法，因为它基于其他state的value进行更新。从第三章中我们知道：
\begin{align\*}
v\_{\pi}(s) & = \mathbb{E}\_{\pi}\left[G_t\right]\tag{3}\\\\
& = \mathbb{E}\_{\pi}\left[R\_{t+1}+\gamma G\_{t+1}| S_t = s\right]\tag{4}\\\\
& = \mathbb{E}\_{\pi}\left[R\_{t+1}+\gamma V(S\_{t+1})|S_t = s\right]\tag{5}\\\\
\end{align\*}
MC使用式子$(3)$的estimate作为target，而DP使用式子$(5)$的estimate作为target。MC是估计是因为式子$(3)$中的expectation是不知道的，用一个sample的return代替真实的expected return；DP是因为$v\_{\pi}(S\_{t+1})$是不知道的，用$V(S\_{t+1})$作为一个估计。TD target也是estimate，TD既进行了采样也使用了$V$的估计值，它对式子$(4)$中的tranisition进行sample，同时使用$v\_{\pi}$的估计值$V$进行计算。所以TD结合了MC的采样以及DP的bootstrap。
![backup_TD](backup_td.png)
TD的backup图如图所示。TD和MC更新叫做sample update，因为这两个算法都牵涉到采样一个successor state，使用这个successor和reward计算一个backup value。sample updates和expected updates的不同在于，他们一个使用sample更新，一个使用所有可能的successors进行更新。
$R\_{t+1} + \gamma V(S\_{t+1}) - V(S_t)$可以看成一类error，度量了$S_t$的estimated value $S_t$和一个更好的estimated value之间的差异$R\_{t+1} +\gamma V(S\_{t+1})$，这叫做$TD error$，用$\delta_t$表示。$\delta_t$是$V(S_t)$的error，在$t+1$时刻可用，如果$V$在一个episdoe中不变的话，就像MC方法一样，那么MC error可以写成TD errors的和。
\begin{align\*}
G_t - V(S_t) & = R\_{t+1} + \gamma G\_{t+1} - V(S_t) + \gamma V(S\_{t+1}) - \gamma V(S\_{t+1})\\\\
& = R\_{t+1} + \gamma V(S\_{t+1}) - V(S_t) + \gamma G\_{t+1} - \gamma V(S\_{t+1})\\\\
& = \delta_t + \gamma G\_{t+1} - \gamma V(S\_{t+1})\\\\
& = \delta_t + \gamma(G\_{t+1} - V(S\_{t+1}))\\\\
& = \delta_t + \gamma\delta\_{t+1} + \gamma\^2(G\_{t+2} - V(S\_{t+2}))\\\\
& = \delta_t + \gamma\delta\_{t+1} + \gamma^2\delta\_{t+2} + \cdots + \gamma^{T-t-1}\delta\_{T-1} + \gamma\^{T-t}(G_T-V(S_T))\\\\
& = \delta_t + \gamma\delta\_{t+1} + \gamma^2\delta\_{t+2} + \cdots + \gamma^{T-t-1}\delta\_{T-1} + \gamma\^{T-t}(0-0)\\\\
& = \sum\_{k=t}\^{T-1} \gamma\^{k-t}\delta_k \tag{6}\\\\
\end{align\*}
如果$V$在一个episode中改变了的话，像$TD(0)$一样，这个公式就不精确成立了，如果$\alpha$足够小的话，还是近似成立的。

\begin{align\*}
G_t - V_t(S_t) & = R\_{t+1} + \gamma G\_{t+1} - V_t(S_t) + \gamma V\_{t+1}(S\_{t+1}) - \gamma V\_{t+1}(S\_{t+1})\\\\
& = R\_{t+1} + \gamma V\_{t+1}(S\_{t+1}) - V_t(S_t) + \gamma G\_{t+1}- \gamma V\_{t+1}(S\_{t+1})\\\\
&= \\\\
\end{align\*}
