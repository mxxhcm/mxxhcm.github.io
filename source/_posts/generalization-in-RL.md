---
title: generalization in RL
date: 2019-09-05 12:25:40
tags:
 - reinforcement learning
 - 强化学习
categories: 强化学习
---

## Gotta Learn Fast: A New Benchmark for Generalization in RL

### 下载地址
https://arxiv.org/pdf/1804.03720.pdf

### 摘要
这篇文章基于Sonic the Hedgehog视频游戏提出了Gym Retro rl benchmark。这个benchmark主要用于评估RL领域的transfer learning和few-shot learning算法。

### 简介
像ALE之类的RL benchmark并不是一个理想的测试相似任务之间泛化性能的benchmark。RL相当于在测试集上进行训练，然后展现出和训练环境相同环境下算法的最终性能。所以，这篇文章给出了一个benchmark，和监督学习一样，区分了train和test环境。
作者给出的benchmark设计了一个meta-learning的RL dataset，从单个任务分布中采样很多个相似的任务，可以作为few-shot RL算法的test bed。除了few-shot learning，这个benchmark还可以用来衡量cross-task generalization，train set和test set的设置可以用来在一些levels训练，然后迁移到另一些levels。
Gym Retro 和Retro Learning Environment以及Arcade Learning Environment都有关，但是，Gym Retro更灵活，更容易扩展，有相当多的environments。OpenAI想要做的是RL中的Omniglot和mini-ImageNet。
在之前RL领域的transfer learning中，主要有两种evaluation techniques，在人工设计的task上进行evaluation，或者在ALE上进行evaluation。人工设计的任务中，许多不同的算法难以进行比较，而ALE中，不同的任务之间差异太大，使用transfer learning不可能获得太大的improvements。

### Sonic benchmark
这一章详细介绍了Sonic benchmark，从technical details到高维的特征设计。
#### Gym Repo
Sonic benchmark的底层是Gym Retro，一个video games仿真RL environments项目。Gym Retro的核心是gym-retro python包，它将各种emulated games都封装成Gym environments，使用统一的接口进行操作。Gym-Retro中的每一个游戏都由一个ROM，一个或多个svae state，一个或多个scenarios以及一个data file组成。
- ROM，组成游戏的data和code，通过一个emulator进行加载。
- Save state，游戏的在某个console状态的截图。
- Data file，描述在console memory中各种各样的信息，比如包含分数在哪里存储。
- Scenario，描述完成条件以及奖励函数。

#### The Sonic Video Game
Sonic benchmark包含三个相似的游戏：Sonic The Hedgehog, Sonic The Hedgehog2,Sonic 3 Knuckles。这三个游戏的规则和操作都很像，当然还是有一些差别。通过使用这几个游戏获得尽可能多的environments作为datasets。
每一个sonic game被划分为zones，每一个zone又进一步划分为acts，相当于我们说的关卡的意思。整个游戏的规则和目的都是一样的，每一个zone都有独特的textures和objects，一个zone中的不同acts share textures和objects，但是空间布局不同。作者把(ROM, zone, act)称为一个level。

#### Games and Levels
整个benchmark包含$3$个游戏的$58$个save states，每一个save states都有different level开始时的player。许多原始游戏中的acts没有被使用，因为它们只包含打boss或者和设计的reward function不兼容。
随机的选择包含超过一个act的zones，然后随机的从选中的zone中选择act。Test set包含绝大数在train sets中出现的objects和objects，但是layouts不同。
Test levels如下表所示：
ROM |one|Act
:--:|:--:|:--:
Sonic The Hedgehog| SpringYardZone| 1
Sonic The Hedgehog| GreenHillZone| 2
Sonic The Hedgehog| StarLightZone| 3
Sonic The Hedgehog| ScrapBrainZone| 1
Sonic The Hedgehog 2| MetropolisZone| 3
Sonic The Hedgehog 2| HillTopZone| 2
Sonic The Hedgehog 2| CasinoNightZone| 2
Sonic 3 & Knuckles| LavaReefZone| 1
Sonic 3 & Knuckles| FlyingBatteryZone| 2
Sonic 3 & Knuckles| HydrocityZone| 1
Sonic 3 & Knuckles| AngelIslandZone| 2

#### Frame Skip
原生的gym-retro environments每隔$\frac{1}{60}$秒调用一次step()方法。在ALE环境的实践中，通常使用长为$4$的fram skip of $4$。使用timesteps作为测量游戏内时间的主要单位，则使用长为$4$的frame skip，一个timesteps代表一秒的$\frac{1}{15}$。而deterministic environments容易受到trivial scripted solutions的影响，作者使用了sticky fram skip，即每一个action有$0.25$的概率延迟一个frame，也就是说这个action aplly了$5$次。

#### Episode Boundaries
游戏中的experience被划分成episodes，对应于存活，在每一个episode的结束，environment重置为save states。每个episode在遇到以下任何一个条件时结束：
- 成功的完成这个level，在这个benchmark中，完成一个level对应于通过这个level内一个确定的水平offset。
- 玩家失败。
- 当前episodes经过了4500个episodes，大概有$300$s，$5$分钟。

如果满足以上任意一个条件，environment都应该被重置，agents不应该使用任何特殊的APIs告诉environments开始一个新的episode。这个benchmark取消了打boss这一环节，这个难度太大了。

#### Observations
Gym-Retro环境在每一个timestep开始前产生一个observation，这个observation是一个三通道的图像，不同的游戏维度不同，对于Sonic，images是$320\times 224$大小的。


## Quantifying Generalization in Reinforcement Learning 
### 下载地址
https://arxiv.org/pdf/1812.02341.pdf

### 摘要

## 参考文献
1.https://arxiv.org/pdf/1804.03720.pdf
2.https://arxiv.org/pdf/1812.02341.pdf
