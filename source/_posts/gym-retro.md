---
title: gym retro
date: 2019-09-15 20:28:55
tags:
 - gym-retro
 - 强化学习
 - gym
categories: gym
---

## Gym Retro
### 什么是Gym Retro？
将不同平台的video games都转换成gym environments。Gym-retro包括了RLE和ALE两个环境，可以使用统一的gym接口进行管理，更灵活，更容易扩展。

### 创建gym retro的目的
此前transfer learning in RL，有两种通用的evaluation：
- 在人工合成的task上进行evaluation
- 在ALE环境上进行evaluation
这两种方式都有缺陷：前者的话不同的算法之间很难比较，后者的话，效果很差，因为不同的游戏之间差异可能很大。
本文的提出的gym-retro可以：
1. 作为few-shot RL的benchmark，从一个single task distribution中sampling许多simlar tasks。
2. 作为transfer learning的benchmark，提供cross-task的dataset。划分train/test让agent学会在some levels进行explore，迁移到其他levels。

### 包含哪些平台
- Atari
- NEC
- Nintndo
- Sega

### 包含哪些ROMs
- the 128 sine-dot by Anthrox
- Sega Tween by Ben Ryves
- Happy 10! by Blind IO
- 512-Colour Test Demo by Chris Covell
- Dekadrive by Dekadence
- Automaton by Derek Ledbetter
- Fire by dox
- FamiCON intro by dr88
- Airstriker by Electrokinesis
- Lost Marbles by Vantage

## Sonic benchmark
### Gym Retro
Sonich benchmark的底层实现是Gym Retro，它可以提供接口给诸如RLE game将其封装成gym environment。
每个game由ROM, 一个或者多个save states，一个或者多个scenarios，还有一个data file。
- ROM：组成game的代码和数据。
- Save state：console's state的截图。
- Data file：描述各种各样的information在console memory的哪个地方存着。
- Scenario：描述done conditions和reward functions。

### Sonic game
Sonic benchmark中有三个相似的游戏：$Sonic The Hedgehog^{TM} $, $Sonic The Hedgehog^{T} 2$, $Sonic 3 Knuckles$，这些游戏具有相似的规则，可能有一些差别，但是不大。
每个Sonic game都有zones，每个zone又被划分成acts。每个zone有独特的纹理和物体；每个zone中的act有相同的问题，但是布局不同。一个(ROM, zone, act)被称为一个level。

### Games and Levels
Sonic benchmark中所有的games总共有$58$个save states，每一个save state对应一个不同的level。
选取test set的时候随机选取超过$1$个act的zones，然后从中随机选取一个act。这样子，test set中巨大部分的纹理和物体都在training set中出现过，只不过layouts不同。所有的test levels如下：
|ROM|Zone|Act|
|:--:|:--:|:--:|
|Sonic The Hedgehog|SpringYardZone|1|
|Sonic The Hedgehog|GreenHillZone|2|
|Sonic The Hedgehog|StarLightZone|3|
|Sonic The Hedgehog|ScrapBrainZone|1|
|Sonic The Hedgehog 2|MetropolisZone|3|
|Sonic The Hedgehog 2|HillTopZone|2|
|Sonic The Hedgehog 2|CasinoNightZone|2|
|Sonic 3 & Knuckles|LavaReefZone|1|
|Sonic 3 & Knuckles|FlyingBatteryZone|2|
|Sonic 3 & Knuckles|HydrocityZone|1|
|Sonic 3 & Knuckles|AngelIslandZone|2|

### Frame skip
step()方法的调用频率是$60$hz，dqn中经常使用的frames skip为$4$，Sonic benchmark也用了，用timestep表示使用了frame skip之后一步的时长，也就是$\frac{4}{60}$。
Sonic benmeark对frame skip做了一些改变，叫做stick frame skip。对于agent的action加了一些随机性：每隔$n$帧之后的那一帧，有$0.25$的概率继续应用之前的那个action。

### Episode boundaries
游戏中的experience划分为episodes，大概对应有几条命。每一个episode结束，重置到相应的save state。每个episodes结束的条件有以下几种：
- 通关，去掉了boss，通过水平方向上一个特定的偏移量就算通关。
- 少了一条命
- 当前episode累计玩了$4500$个timesteps，大概是$5$分钟。

### Observations
24位RBG图像，Sonic是$320 \times 224 $

### Actions
所有的Sega Genesis游戏的action space包含：
B, A, MODE, START, UP, DOWN, LEFT, RIGHT, C, Y, X, Z
Sonic game中有八个很重要的buttion combinations:
{ {}, {LEFT},  {RIGHT}, {LEFT, DOWN}, {RIGHT, DOWN}, {DOWN}, {DOWN, B}, {B} }

### Rewards
在一个episode中，cumulative reward和玩家的初始位置到当前位置的偏移是成正比的。也就是说往右走产生正的reward，往左走产生负的reward。
Reward由horimontal offset和completion bonus构成。Completion bonus是$1000$，在$4500$个timesteps内线性降为$0$。
但是imediate reward可能是有欺骗性的，有时候为了获得更大的reward往回走是必须的。

### Evaluation
使用test set中所有levels的mean score作为metric。步骤入如下：
1. 训练时候，使用任意数量的训练集（你喜欢多少就用多少）训练
2. 测试的时候，每一个test levels执行$100$万个timesteps。在test每一个level的时候都是分开的。
3. 对每个level的$100$万个timesteps中所有episode的total rewards进行平均。
4. 对所有test level的scores进行平均。

这个$100$万是怎么选出来的？在无限个timestep的场景下，没有证明meta-learning或者transfer-learning是必要的，但是在有限个timestep的场景下，transfer learning对于得到好的performance是很必要的。


## Baselines
- Humans：四个玩家，每个玩家在training levels上训练两个小时。然后在每个test level玩一个小时。
- RainBow：设置$V\_{max} = 200$，replay buffer从$1M$改成了$0.5$M，直接使用Rainbow的初值。Action space：{ {LEFT}, {RIGHT}, {LEFT, DOWN}, {RIGHT, DOWN}, {DOWN}, {DOWN, B}, {B} }。Agent的reward是基于agent到过的最大$x$，这样子不会惩罚它往回走。
- JERK：并没有使用depp learning，叫JERK(Just Enough Retained Knowledge)。使用一个简单的算法进行explore，然后回放训练过程中的best action。因为环境是stochasitc，不知道哪个action是最好的，因此次他仅仅是一个mean。
- PPO：在每个test levels上，单独的调用PPO。和Rainbow的action，observation spaces一样，CNN架构和ppo论文中一样。超参数：
Hyper-parameter|Value
:-:|:-:
Workers|1
Horizon|8192
Epochs|4
Minibatch size|8192
Discount ($\gamma$) | 0.99
GAE parameter ($\lambda$) | 0.95
Clipping parameter ($\epsilon$)|0.2
Entropy coeff.|0.001
Reward scale|0.005
- Joint PPO：从training levels迁移到test levels，在training levels上训练一个policy，然后用它作为test levels的初始化。
在meta-learning过程中，训练一个single-policy玩training set中的每一个level。具体的，运行$188$个parallel works，每个worker运行training set中的一个level。在每一个gradient step，对所有workers的gradients进行平均，确保policy在整个training set上的训练是平稳的。整个过程需要几百个millions timesteps才会收敛。参数设置和PPO一样。
在所有training sets上完成训练以后，使用这个网络在作为test set中网络的初始化，除了使用meta-learning的结果进行初始化，所有的其他过程和PPO一样。
- Joint Rainbow：和Joint PPO的训练过程一样。
没有joint training的Rainbow超过了PPO，但是joint Rainbow没有超过joint PPO。在整个训练集上训练单个Rainbow的时候，使用了$32$个GPUs。每一个GPU对应一个单个的worker，每一个worker有自己的replay buffer和$8$个环境。环境也是joint environments，在每一个episode开始的时候采样一个新的tranining level。
除了batch size和distributed worker，其他的超参数和常规的Rainbow一样。


## 示例
### 安装gym retro
``` shell
pip3 install gym-retro
```

### 创建Gym Env
下面代码创建一个gym环境
``` python
import retro
env = retro.make(game='Airstriker-Genesis')
```
retro中的environment是从gym.Env类继承而来的。

### 默认ROM
Airstriker-Genesis的ROM是默认包含在gym-retro之中的，其他的一些ROMs需要手动添加。

### 所有的games
``` python
import retro
retro.data.list_games()
```
上述代码会列出所有的游戏，包含那些默认没有集成的ROMS中的。

### 手动添加ROMs
``` shell
python3 -m retor.import /path/to/your/ROMs/directory
```
上述代码将存放在某个路径下的ROMs拷贝到Gym Retro的集成目录中去。

## 参考文献
1.https://retro.readthedocs.io/en/latest/
2.https://arxiv.org/pdf/1804.03720.pdf

