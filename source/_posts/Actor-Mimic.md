---
title: Actor-Mimic
date: 2019-10-14 11:43:48
tags:
 - 强化学习
 - rl papers
categories: 强化学习
mathjax: true
---

## Actor Mimic
本文提出了Actor-Mimic，一个multitask和transfer learning方法，使用多个expert DQN指导训练一个可以在多个taskes上使用的单个policy network，并且可以将这些经验迁移到新的taskes上。

## Policy Regression Objective
给定多个sources games $S_1, \cdots, S_N$，我们的第一个目标是获得一个能玩任何source games，并且尽可能和expert DQN性能相近的single multitask policy network。为了训练这样一个网络，使用$N$个expert DQN $E_1, \cdots, E_N$进行指导。一个可能的方法是定义student network和expert network之间$Q$值的均方根误差。因为expert values funcitons在不同的游戏之间可能变化很大，所以作者首先将$Q$值经过softmax变成了policies，softmax的输出都在$0$和$1$之间，所以可以提高训练的稳定性。我们可以把softmax看成让student更多的关注expert DQN在每个state选择的action（DQN选择的是Q值最大的action），经过softmax相当于让它更sharp了。
最后得到了一个actor，或者说是一个policy，它模仿了所有DQN experts的decisions。比如，在$Q$值上计算Boltzman分布：
$$\pi{\_{E\_i}} (a|s) = frac{e^{\tau^{-1} Q\_{E_i}(s,a) } }{\sum\_{a'\in A\_{E_i} } e^{\tau^{-1} Q\_{E_i}(s,a) } } \tag{1}$$
其中$\tau$是温度，$A\_{E_i}$是expert DQN $E_i$使用的action space。给定$S_i$的一个state s，定义multitask  network的policy objective是expert network's policy和currnet multitask policcy的cross-entropy:
$$L^i\_{policy}(\theta) = \sum\_{a\in A\_{E_i} }\pi\_{E_i} (a|s) \log \pi\_{AMN}(a|s;\theta) $$
其中$\pi\_{AMN}(a|s;\theta) $是$\theta$参数化的multitask Actor Mimic Network policy。和Q-learning把自身当做target value相比，AMN得到了一个stable supervised training signal (expert network)指导 multitask network训练。
为了获得训练数据，可以sample expert network后者使用AMN action outputs生成trajectories。即使AMN还在学习过程中，也能得到好的结果。至少在AMN是linear function approximator时，可以证明AMN会收敛到expert policy。

## Feature Regression Objective
除了对policy进行回归以外，还可以对feature进行回归。用$h\_{AMN}(s)$和$h\_{E\_i}(s)$分别表示AMN和第$i$个expert network在state s处feature的hidden activation，他们两个的dimension不一定要相等。使用一个feature回归网络$f_i(h\_{AMN}(s))$，预测$s$处$h\_{E_i}(s)$到$h\_{AMN}(s)$的映射，映射$f_i$的结构是随意的，可以使用以下的回归loss进行训练：
$$L^i\_{FeatureRegression}(\theta, \theta\_{f_i}) = || f_i(h\_{AMN}(s;\theta); \theta\_{f_i}) - h\_{E_i}(s) ||^2_2 \tag{}$$
其中$\theta$是AMN的参数，$\theta\_{f_i}$是第$i$个特征回归网络的参数。使用这个loss训练，最终我们的目标是得到一个multitask network能够包含多个expert network的features。

## Actor-Mimic Objective
将policy objective和feature objective结合在一起，就得到了actor-mimic objective：
$$ L^i\_{ActorMimic}(\theta, \theta\_{f_i}) = L^i\_{policy}(\theta) + \beta L^i\_{FeatureRegression}(\theta, \theta\_{f_i}) \tag{]$$
$\beta$用来控制两个objective的权重。直观上来说，我们可以把policy objective看成expert network教会AMN该怎么act（模仿expert的action），而feature objective类似于expert network教会AMN为什么这样act，模仿expert的思考过程（特征提取过程）。

## Transfering Knowledge
通过优化actor-mimic objective，我们得到一个在所有source target上都表现不错的expert network，接下来我们可以把它迁移到相关的target task上。为了迁移到新的task上，首先移除掉AMN的final softmax layer，然后用AMN的参数初始化一个DQN在新的task上继续训练，接下来和标准的DQN训练方式一样。Multitask pretaining可以看成学习了related tasks中对于policies definition相当有效的特征，然后初始化DQN。如果source和target tasks很像的话，pretained features对于target task是相当有效的。

## 参考文献
1.
