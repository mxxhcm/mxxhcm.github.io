---
title: CoinRun
date: 2019-09-05 12:25:40
tags:
 - reinforcement learning
 - 强化学习
categories: 强化学习
---

## Quantifying Generalization in Reinforcement Learning 
### 下载地址
https://arxiv.org/pdf/1812.02341.pdf

### 摘要
这篇文章研究的是drl中的overfitting问题。在绝大多数的RL benchmark中，训练和测试都习惯性的在相同环境中，这就让agent's的泛化能力不够优秀。本文通过程序生成环境构建不同的training set和testing set解决这个问题，设计了一个新的CoinRun环境用于RL的generalization。
实验证明更深的cnn能够改善generalization，监督学习中常用的一些方法，如$L2$正则化，dropout，data augmentation以及batch normalization。
这篇文章的贡献：
1. transfer需要的训练环境数量要远远高于之前的工作中使用到的。
2. 使用CoinRun benchmark提出了一个generalization metric。
3. 找到了不同网络结构和正则化项中最好的那些设置。

### Related Work
这篇文章是从Sonic benchmark中得到的idea，agents可以在training set上训练任意的长度，在test时候执行$1$ million timesteps进行fine-tuning，sonic benchmark的目标是解决"training on the test set"的问题。


### 简介
尽管RL agents能解决很复杂的任务，将experience迁移到新环境，或者在不同的RL tasks之间进行泛化是很困难的。即使已经掌握了$10$ video game的agents，在初次遇到第$11$级时也会失败，agents在训练的时候只学习到了和这个环境相关的知识。
RL agents的训练其实是过拟合的，但是绝大多数的benchmark还是鼓励在相同的环境中进行train和evaluation。和Sonic Benchmark一样，作者认为分train和test集是必要的，同时作者认为量化agent的泛化能力也是很有用的。
Sonic Benchmark通过对环境划分训练集和测试集，解决了在测试集上训练的问题。而Farebrother认识到，没有划分训练集和测试集，使得RL训练中缺乏正则化手段，通过使用监督学习中的$L2$正则化和dropout，使得agents能够学习更多泛化特征。
Zhang等利用程序生成划分train和test环境，他们在gridworld mase上进行实验，得出了许多RL agents为什么过拟合的结论。通过让agents记住在训练集中具体的levels，以及stick actions，random starts，可以减少过拟合的发生。

### Quantifying Generalization
#### CoinRun
CoinRun环境很简单，一个智能体，从最左边，一直往右走，收集在最右边的金币，中间或均匀或随机的散布着一些障碍，只有得到金币后有一个常数的reward。一个episode结束有以下三种可能性：
- 当agent死亡，
- 或者采集到coin，
- 或者$1000$个timesteps后自动结束。

CoinRun和Sonic很像，但是更简单，更容易泛化。给定足够数量的training levels和足够的training time，算法能够学习一个接近最优的optimal policy for all CointRun levels。每一个level都是从一个给定的seed中顺序生成的，能够给智能体相当多并且易于量化提供的training data。

#### CoinRun泛化曲线
使用CoinRun可以衡量agents从给定的training level中训练迁移到没有见过的test levels上的效果。随着training中使用的levels数量增多，即使是训练固定的timesteps时，我们期望在test set上的性能变好。test时，作者在test set上进行了zero-shot performance评估，即没有在test set上进行fine-tuning。
作者训练了$9$个agents 运行CoinRun，每一个都在具有不同levels数量的训练集上运行。训练过程中，每一个episode从相应的set中随机采样一个level。前$8$个agents使用的train sets中level的数量从$100$到$16000$，最后一个agent在无限个levels的训练集上。Level的seed是$2^32$，所以几乎不可能发生冲突。训练集包含$2$M个独一无二的levels，在test的时候仍然不会遇到已经遇到过的level。整个实验进行了$5$次，每次重新生成training sets。
使用nature-dqn中的三层CNN进行训练，然后使用$8$个workers的PPO总共训练了$256M$步。每个agents都训练相同的timesteps，然后在每个mini-batch的$8$个works上取gradients的均值。
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

## 其他环境
### CointRun-Platforms
### RandomMazes


## 参考文献
1.https://arxiv.org/pdf/1804.03720.pdf
2.https://arxiv.org/pdf/1812.02341.pdf
