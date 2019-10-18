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


## 实验
- single game policy distillation:
四个游戏，四个网络：dqn expert, distill-MSE, distill-NLL,distill-KL，四个网络的大小都和nature DQN一样。
- single game policy distillation with compression
十个游戏，四个网络：dqn expert, $25\\%$ distill-KL，$7\\%$ distill-KL，$4\\%$ distill-KL，后面三个网络大小分别是dqn expert的$25\\%, 7\\%, 4\\%$。
- multi-task distillation
三个游戏，三个网络：multi-dqn, multi-dist-NLL, multi-dist-KL，这三个网络的大小都和nature dqn一样。
十个游戏，一个网络：multi-dist-KL，大小是nature dqn的4倍。
- online policy distillation

Single game policy distillation实验中，teacher network是一个已经训练完成的model，选择一个DQN expert作为teacher network，训练student network时，teacher network不进行Q-learning，只是用来采样，相当于产生监督学习的样本。Student network学习teacher network是怎么将输入和label对应的。Teacher network的输入(images)和输出(Q值）都被存在buffer中。Multitask policy distillation的训练过程类似。
除了模型压缩时候用到的DQN，以及一个$10$个games的multi-task distillation任务中用到的DQN，它的参数比nature DQN多四倍还有额外的fully connected layer，所有其他的DQN都和nature DQN的结构一样。
评价指标用的是Double DQN中的normalized score。

### single game policy distillation
在这个实验中，作者测试了single game的distillation，将一个DQN expert的knowledge迁移到一个新的结构相同的随机初始化的DQN。分别使用了三种loss：MSE, NLL，KL散度进行训练。结构证明KL好于NLL好于MSE。
原因分析：
MSE是因为$Q$值在一定范围内，MSE loss都会很小，如果某个state处不同action的Q值很接近的话，即使MSE很小，也会产生误差。
NLL loss假设每次只有一个optimal action，原则上没有错。但是我们的teacher network可能不是optimal，最小化NLL的过程可能将一些noise也进行了变化。

### policy distillation with model compression
这一节介绍的是policy distillation model compression。训练的时候，模型大一些有助于训练，但是训练好的模型进行压缩也保留性能。　
分别在$10$个不同的atarti游戏上进行single-game distilled，使用的都是KL loss，student分别压缩为teacher的$25\\%, 7\\%, 4\\%$，压缩到$25\\%$ student network的平均性能是teacher network的$108\\%$,压缩到$25\\%$ student network的平均性能是teacher network的$102\\%$, 压缩到$25\\%$ student network的$4\\%$的平均性能是teacher network的$84\\%$。
single policy distillation with model compression中网络结构：
Agent | Input | Conv. 1 | Conv. 2 | Conv. 3 | F.C. 1 | Output | Parameters
Teacher (DQN) | 4 |  32 | 64 | 64 | 512 | up to 18 | 1,693,362
Dist-KL-net1 | 4 | 16 | 32 | 32 | 256 | up to 18 | 427,874
Dist-KL-net2 | 4 | 16 | 16 | 16 | 128 | up to 18 | 113,346
Dist-KL-net3 | 4 | 16 | 16 | 16 | 64 | up to 18 | 61,954
模型压缩只改变了参数的数量，没有改变模型架构。

### multi-game policy distillation
Multi-task DQN是multi-task distillation的baseline，实验使用了三个游戏，multi-task DQN和单个DQN的训练过程一样，但是使用了三个游戏的experient进行训练。对比了multi task DQN，multi distillation NLL，multi distillation KL，他们的网络大小都是一样的。
最后作者还将$10$个游戏distill到一个single student network中，这个network大小是nature DQN的四倍。
multi-task distilltaion experiments中网络结构：
Agent | Input | Conv. 1 | Conv. 2 | Conv. 3 | F.C. 1 | F.C. 2 | Output | Parameters
One Teacher (DQN) | 4 | 32 | 64 | 64 | 512 | n/a | up to 18 | 1,693,362
Multi-DQN/Dist (3 games) | 4 | 32 | 64 | 64 | 512 | 128 (x3) | up to 18 (x3) | 1,882,668
Multi-Dist-KL (10 games) | 4 | 64 | 64 | 64 | 1500 | 128 (x10) | up to 18 (x10) | 6,756,721

### online policy distillation

## Experimental Details
### Policy Distillation Training
Policy distillation online data collection和nature DQN中agent evaluation一样，DQN随机执行最多$30$个null-ops初始化episode，使用$\epsilon$-greedy($\epsilon=0.05$)算法进行$30$分钟即$108000$frames的evaluation。
DQN expert的输入是图像，输出是$Q$值，replay buffer记录$10$个小时的experience（$15$Hz下共$54000$个control steps），

### Distillation Targets

### Agent Evaluation
使用human starts，使用$\epsilon$-greedy($\epsilon=0.05$)算法进行$30$分钟即$108000$frames的evaluation。
在multitask中，使用$\frac{\text{student score}}{\text{DQN score}}$当做metric。

## 代码
官方没有放出代码，有其他人的复现版本：
https://github.com/ciwang/policydistillation

## 参考文献
1.https://arxiv.org/pdf/1511.06295.pdf
