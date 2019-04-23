## 问题
本文解决的问题是一种形式的迁移学习，称为domain adaptation。Domain adaptation场景中，source domain上，一个智能体在具有特殊reward结构的input distribution上进行训练，在target domain上，input distribution变了，但是reward structure没有变。
本文的目标是仅仅在source domain 上进行学习，学习到的策略直接推广到target domain，性能下降很小。

## 意义
1 之前的一些做法是对齐source domain和target domain的分布，但是这种方法对于target domain的信息依赖是一个问题，因为数据是很昂贵的，一般很难获得，甚至根本就不能提前知道target domain。
2 仅仅在source domain学习到的策略容易在source input distribution过拟合，造成domain adaptation performance的下降。


## 方法
本文的想法是，假设有一个MDP set M包含了自然界中所有的MDPs，我们用到的每一个MDP $D_i$都是从这个集合M中采样得到的。这个集合M同时是一个状态空间$\mathcal{S}$，状态空间$\mathcal{S}$中是每一个MDP $D_i$观测值的生成因子，整个集合M中的MDP都是由这些因子生成的。即任意的一个MDP $D_i$的状态空间S都是$\mathcal{S}$的子集。

本文提出的方法总共分为三步。第一步利用监督学习方法$\beta-VAE$学习出source domain的低维特征表示（这里就是假设source domain和target domain中所有的输入分布都是由这些低维特征组成的）；第二步利用已有的强化学习方法，如DQN，A3C等，学习出source domain 上的策略，输入是第一步中学习到的低维特征表示，输出是学习到的策略。第三步将学习到的策略迁移到target domain，不做fine-tuning。

## 例子
source domain：橙子在蓝色的屋子里，苹果在红色的屋子里。
target domain：橙子在红色的屋子里，苹果在蓝色的屋子里。

智能体的目标是拿到橙子，避免苹果。

在source domain上的学习是，输入->隐藏状态->动作。
智能体很有可能会基于橙子/蓝屋子->good，苹果/红屋子->bad。做出相应的决策，这就会造成过拟合。

而我们想要在source domain中学到的是，橙子->good,apple->bad。
我们想要做到的是在source domain中学到的特征在source domain和target domain都成立。
我们把source domain中的观测进行分解：
room:蓝。 object：橙子。

## 实验

1 DeepMind Lab
两种颜色的屋子，四种类型的物体。
在每个屋子里会有两个物体，一个是good，一个是bad。

2 Jaco Arm and MuJoCo
sim2sim
target domain的input distribution和source domain重叠很少。
sim2real
target domain的采样分布和source domain一样，但是很多现实世界中的细节在sim中没有实现，如阴影，多光源等等，理想-现实差距。