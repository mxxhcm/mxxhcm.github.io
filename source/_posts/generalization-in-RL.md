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

#### Actions

#### Rewards
Reward由两部分组成：一个水平offset和一个完成任务的bonus，每个level的horizontal offset被归一化了，如果它通过了作为终点的那个horizontal offset，每个agent总的reward是$9000$。这样子便于不同长度的level之间进行比较。完成的bonus是$1000$，在$4500$个timesteps内线性减少为$0$，这是为了鼓励agents尽可能快的完成这个游戏。


#### Evaluation
使用test set中所有levels的mean score作为metric。步骤入如下：
1. 训练时候，使用尽可能少的训练集训练
2. 测试的时候，每一个test levels执行$100$万个timesteps。在test每一个level的时候都是分开的。
3. 对每个level的$100$万个timesteps中所有episode的total rewards进行平均。
4. 对所有test level的scores进行平均。

这个$100$万是怎么选出来的？在无限个timestep的场景下，没有证明meta-learning或者transfer-learning是必要的，但是在有限个timestep的场景下，transfer learning对于得到好的performance是很必要的。

### Baselines
作者给出了几个方法在benchmark上的效果，包含人类，没有使用训练集的一些方法以及使用joint training加上fine tunning的一些迁移学习方法。

1. 人类
有四个人，每个人先在训练集上玩两个小时，然后在每个test level上玩一个小时。
2. Rainbow
Rainbow是DQN的变种，加了各种各样的trick。
这里对Rainbow做了一些改变：设置$V_{max} = 200$；buffer size为$0.5M$；直接使用了rainbow中的参数初始化方式，没有系列化实验。
3. JERK
4. PPO
每一个PPO单独运行在一个test level上。action和observation space都和rainbow一样，相同的reward preprocessing。同时还使用了一个小的常数对reward进行缩放。CNN结构和dqn一样。
5. Joint PPO
在所有的training levels上训练一个policy，然后用它作为test levels的初值。
在训练过程中，训练一个policy，使用188个parallel的workers，每一个都对应training set中的一个level。在每一gradient step，所有的workers平均梯度，确保policy是在整个训练集上得到的。整个训练过程需要几百万个timesteps才收敛，除了训练的setting不同，其他参数和常规的PPO一样。
6. Joint Rainbow
没有joint training的Rainbow超过了PPO，但是joint Rainbow没有超过joint PPO。在整个训练集上训练单个Rainbow的时候，使用了$32$个GPUs。每一个GPU对应一个单个的worker，每一个worker有自己的replay buffer和$8$个环境。环境也是joint environments，在每一个episode开始的时候采样一个新的tranining level。
除了batch size和distributed worker，其他的超参数和常规的Rainbow一样。

### 结果
本文提出了一个benchmark，并用这个benchmark评估了几个baselines。结果表明最好的transfer结果没有比重头开始训练的结果好多少。而且离最好的的score（设计时是$9000$到$10000$）有一定距离。


## Quantifying Generalization in Reinforcement Learning 
### 下载地址
https://arxiv.org/pdf/1812.02341.pdf

### 摘要
这篇文章研究的是drl中的overfitting问题。在绝大多数的RL benchmark中，训练和测试都习惯性的在相同环境中，这就让agent's的泛化能力不够优秀。本文通过程序生成环境构建不同的训练集和测试集解决这个问题。本文设计了一个新的CoinRun环境用于RL的generalization。
作者实验证明更深的cnn能够改善generalization，监督学习中常用的一些方法，如$L2$正则化，dropout，data augmentation以及batch normalization。
这篇文章的贡献：
1. transfer需要的训练环境数量要远远高于之前的工作中使用到的。
2. 使用CoinRun benchmark提出了一个generalization metric。
3. 找到了不同网络结构和正则化项中最好的那些设置。

### 简介
尽管RL agents能解决很复杂的任务，将experience迁移到新环境，或者在不同的RL tasks之间进行泛化是很困难的。即使已经掌握了$10$ video game的agents，在初次遇到第$11$级时也会失败，agents在训练的时候只学习到了和这个环境相关的知识。
RL agents的训练其实是过拟合的，但是绝大多数的benchmark还是鼓励在相同的环境中进行train和evaluation。和Sonic Benchmark一样，作者认为分train和test集是必要的，同时作者认为量化agent的泛化能力也是很有用的。
Sonic Benchmark通过对环境划分训练集和测试集，解决了在测试集上训练的问题。而Farebrother认识到，没有划分训练集和测试集，使得RL训练中缺乏正则化手段，通过使用监督学习中的$L2$正则化和dropout，使得agents能够学习更多泛化特征。
Zhang等利用程序生成划分train和test环境，他们在gridworld mase上进行实验，得出了许多RL agents为什么过拟合的结论。通过让agents记住在训练集中具体的levels，以及stick actions，random starts，可以减少过拟合的发生。

### Quantifying Generalization
#### CoinRun
CoinRun环境很简单，一个智能体，从最左边，一直往右走，收集在最右边的金币，中间或均匀或随机的散布着一些障碍，只有得到金币后有一个常数的reward。当agent死亡，或者采集到金币，或者$1000$个timesteps后自动结束。
CoinRun和Sonic很像，但是更简单，更容易泛化。每一个level都是从一个给定的seed中顺序生成的，能够给智能体相当多并且易于量化提供的训练数据。不同的level之间难度差异很大，所以levels的分布相当于agents的学习课程。
#### CoinRun泛化曲线
使用CoinRun可以衡量agents从给定的training level中训练迁移到没有见过的test levels上的效果。随着training中使用的levels数量增多，我们期望在test set上的性能变好，即使是训练固定的timesteps时。test时，作者在test set上进行了zero-shot performance评估，即没有在test set上进行fine-tuning。
作者训练了$9$个agents 运行CoinRun，每一个都在具有不同levels数量的训练集上运行。训练过程中，每一个episode从相应的set中随机采样一个level。前$8$个agents使用的train sets中level的数量从$100$到$16000$，最后一个agent在无限个levels的训练集上。Level的seed是$2^32$，所以几乎不可能发生冲突。训练集包含$2M$个独一无二的levels，在test的时候仍然不会遇到已经遇到过的level。整个实验进行了$5$次，每次重新生成training sets。
首先使用nature-dqn中的三层CNN进行训练，然后使用$8$个workers的PPO总共训练了$256M$步。每个agents都训练相同的timesteps，然后在每个mini-batch的$8$个works上取gradients的均值。
最后的结果中，对$10000$个episodes的final performance进行平均，我们可以看出来，在$4000$个levels以内，过拟合很严重，即使是$1600$个training levels，过拟合也是很多的。当agents遇到的都是新的levels时，表现最好。

### Evaluating Architectures
比较Nature-CNN和IMPALA-CNN，结果证明IMPALA-CNN要好一些。为了评估generalization performance，可以直接在无限个levels的train set上训练agents，然后比较learning curves。在这个settings中，智能体不可能过拟合某一些levels的子集，每一个level都是新的。因为Impala-cnn的记过更好，所以作者尝试了更深的网络，发现效果更好，新的网络使用了$5$个residual blocks，每层的channels数量是原来的两倍。再进一步增加网络深度的时候，发现了returns逐渐变小，同时花费的时间也更多了。
当然，使用无限的training set并不会总是会带来性能的提升。选择好的超参数能够加快训练速度，但是并不一定会改善generalization的性能。通常来讲，在固定levels的训练集上训练，能产生一个更有用的metric。

### Evaluating Regularization
正则化在监督学习中有很重要的作用，因为监督学习更关心的是generalization。监督学习的数据集通常分为训练集和数据集，有很多正则化技巧可以用来减少generalization gap。但是因为RL中训练集和测试集通常是同一个，所以正则化技术就没什么效果。
现在既然RL中要衡量generalization，可能正则化技术就能起到作用。作者分别在CoinRun environment中试了$L2$正则化，dropout，data augmentation以及batch normalization。
这一节中，所有的agents都是在$500$个levels的CoinRun环境中进行的。我们可以看到有过拟合发生，所以就希望这个setting能够evaluating出不同正则化技术的效果。所有接下来实验的均值和方差都是runs$3$次得到的。使用的是有$3$个残差块的IMPALA-CNN。

#### Dropout和L2正则化
作者分别试了dropout为$p\in [0, 0.05, 0.10, 0.15, 0.20, 0.25]$，以及L2正则化权重$w\in [0, 0.5, 1.0, 1.5, 2.0, 2.5]\times 10^{-1} $，一次只试了一个。最终找到了$p=0.1$以及$w=10^{-4} $。使用$L2$正则化训练了$256M$ timesteps，使用dropout训练了$512M$ timesteps，dropout更难收敛，而且效果没有$L2$正则化好用。

#### Data Augmentation
监督学习中，数据增强的手段主要用于图像，包括变换，旋转，亮度调整，锐化等等。不同的数据集可能需要使用不同的augmentations。这里作用使用的是Cutout的一个变形。对于每一个observation，可变大小的矩阵区别被masked掉，这些masked的区域给一个随机的颜色，这个和domain randomization非常相似，用于机器人从仿真到真实世界的transfer。

#### Batch Normalization
在IMPALA-CNN架构中每层CNN之后使用batch normalization。

#### Stochasticity
这个随机性很有意思哦。作者考虑了两种方法，一种是改变环境的stochasticity，另一个是改变policy的stochasticity。首先，使用$\epsilon$-greedy action selection算法向环境中加入stochasticity，在每一个timestep中，有$\epsilon$的概率，使用random action代替policy的action。在之前的一些研究中，$\epsilon$-greedy用来鼓励exploration和防止overfitting。然后，通过改变PPO中的entropy bonus控制policy stochasticity。Baseline中使用的entropy bonus是$k_H = 0.01$。
同时增加训练时间步到$512M$个timesteps。单独向环境或者policy加入stochasticity都能改善generalization。
#### Combining Regularization Methods
作者尝试将data augmentation,batch normalization和L2结合起来，结果表明比任意单独的一种都要好，但是提升的大小并不是很大，可以说这三种方法解决的过拟合问题差不多。
由于某些未知的原因，将stochastic和正则化手段结合起来的效果不理想。

## 参考文献
1.https://arxiv.org/pdf/1804.03720.pdf
2.https://arxiv.org/pdf/1812.02341.pdf
