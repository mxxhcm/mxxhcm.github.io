---
title: Policy Distillation
date: 2019-10-13 14:45:01
tags:
 - 强化学习
 - value-based
categories: 强化学习
mathjax: true
---

## 简介
Policy Distillation可以extract a policy到一个参数更少更高效的model；还可以将多个任务的policy提取到一个model中。作者使用的基本算法是DQN，DQN既作为baseline和distilled policy的性能进行比较，同时也使用DQN作为teacher用于policy distillation。
一般来说，distillation应用在网络输出为概率的情况。DQN中，网络输出的是real-valued and unbounded的action value。当多个actions的$Q$值接近时，很难选择那个action，而某些actions的$Q$值很大时，很容易进行选择。
Policy distillation的优势：
1. 将网络大小压缩到原来的$\frac{1}{15}$，而不损失性能。
2. 多个expert polices可以用一个单独的multi-task policy表示。
3. 可以看成一个real-time online learning process连续的提炼best policy到一个target network，因此可以高效的记录$Q$-learning policy的进化过程。

所有的contributions可以总结为：
1. single game distillation
2. single game distillation with highly compressed models
3. multi-game distillation
4. online distillation

## Single Task Policy Distillation
Distillation将teacher model T的knowledge进行迁移，得到一个参数更少更加高效的student model S。分类网络中distillation的目标通常是将teacher network layer的最后一层传入softmax layer，使用回归学习student model S的参数。
而本节介绍的single task policy distillation，是对$Q$函数而不是对classifier进行transfer，会面临以下问题：
- 一方面，Q是unbounded and unstable，所以它的scale很难确定。此外，计算一个fixed policy的Q值需要很大的计算量。
- 另一方面，让S只预测一个single best action也可能会出现问题，可能有很多actions的Q值接近。

给定teach model T，用它生成大小为$N$的样本集合$D^T = \left[(s_i, \mathbf{q}\_i)\right]\_{i=0}^T $，每一个样本是$s_i$和$\mathbf{q}\_i$，$s_i$是一个observation，$\mathbf{q}\_i$是对应$s_i$处每一个action的$q$值向量。
作者给出了三种policy distillation方法。如下所示：
### Negative log likelyhood loss (NLL)
第一种方法使用teacher中具有最大$Q$值的action $a\_{i,best} = \arg\max(\mathbf{q}\_i$，使用负的log似然loss训练student model $S$直接预测action：
$$L\_{NLL} (D^T, \theta\_{S}) = - \sum\_{i=1}^{\vert D\vert} \log P(a_i=a\_{i,best} | x_i, \theta\_S)\tag{1}$$

### Mean squared error loss (MSE)
第二种方法计算S和T中$Q$值的mse loss：
$$L\_{MSE} (D^T, \theta\_{S}) = - \sum\_{i=1}^{\vert D\vert} || \mathbf{q}\_i^T - \mathbf{q}\_i^S ||^2_2 \tag{2}$$
这种方法在student model中保留每个action的所有$Q$值。

### KL divergence
第三种方法将$Q$值输入softmax layer，相当于求了policy，然后计算S和T的KL散度：
$$L\_{KL} (D^T, \theta\_{S}) = - \sum\_{i=1}^{\vert D\vert} softmax(\frac{\mathbf{q}\_i^T }{\tau})\log \frac{softmax(\frac{\mathbf{q}\_i^T}{\tau}) }{softmax(\mathbf{q}\_i^T) }\tag{3}$$
在传统的分类问题中，$\mathbf{q}^T $的输出是一个peaked distribution，可以通过提高softmax的温度进行soften将更多的信息transfer到student model。
而在policy distillation中，teacher的输出不是一个distribution，而是每个state下所有可能actions的$q$值，我们的目的不是soften它们，而是想要让它们更sharper。
这个和actor-mimic中的policy regressive objective是不是一样。

## Multi-Task Policy Distillation
上面介绍的是单个任务的distillation，这一节介绍multi-task distillation。multi task distillation和single task distallation的过程一样，只不过在中multi task的distillation使用$n$个单独训练完成的DQN experts，使用这$n$个task上的DQN experts distill一个student model，每一个episode切换一个task。因为不同的tasks可能有不同的action sets，每一个task都有一个单独的output layer。在multitask中使用了KL和NLL loss。
这篇文章还对比了multi-task DQN agents和multi-task distillation agents的性能，Multi task DQN是训练一个network同时玩多个游戏，但是没有DQN exoerts的指导。Multi-task DQN和single-game learning的过程类似，不断的优化网络参数，预测给定state处action的$q$值。和multi-task distillation过程一样，每一个episode切换一个task，每一个task有单独的buffer，在每一个task之间不断的交错训练，并且每一个task有单独的output layer。但是multi-task DQN agents无法达到单个DQN expert的性能。可能是因为在训练过程中，不同task之间policy,reward等的相互干扰。

Multi-task distillation和multi-task DQN之间的区别：
- multi-task distillation使用了$n$个DQN expert，即已经训练好的在单个task上都表现不错model，使用他们distill一个新的model。
- multi-task distillation是用一个model回归拟合$n$个model。
- multi-task learning没有使用DQN expert，而是使用一个model去玩$n$个游戏。
- multi-task learning 是train。

Policy distillation可能提供了一种方式将多个polices组合到一个model中而不损害performance，在distillation process中，policy被压缩并且refined了。

## 代码
官方没有放出代码，有其他人的复现版本：
https://github.com/ciwang/policydistillation

## 参考文献
1.https://arxiv.org/pdf/1511.06295.pdf
