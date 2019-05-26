---
title: DQN
date: 2019-03-02 19:29:35
tags: 
 - 强化学习
 - 值迭代
 - 深度学习
 - 机器学习
categories: 强化学习
mathjax: true
---

## Playing Atari with Deep Reinforcement Learning
### 概述
本文首次成功的将深度学习模型应用到强化学习领域，直接从高维的感知输入中学习控制策略。作者提出用一个卷积神经网络来表示值函数，用Q-learning算法的变种来训练这个神经网络。神经网络的输入是原始的图片信息，输出是这个图片对应状态的估计值函数。
Atari 2600是一个强化学习的benchmark，共有2600个游戏，每个智能体会得到一个高维的图像输入(210 x 160 RGB视频60Hz)。本文的目标是设计出一个神经网络能够尽可能成功的学会更多的游戏，这个网络的输入只有视频信息，reward和terminal信号以及可能采取的action，和人类玩家得到的信息是一模一样的，当然是计算机能看得懂的信号。

### 存在的问题
1. 目前绝大多数深度学习问题都需要大量有标记的训练数据。
2. 强化学习需要从一个稀疏的，有噪音的，通常是time-delayed(延迟的)标量信号中学习。这个延迟存在于action和reward之间，而且可以达到几千个时间步那么远，和监督学习中输入和输入之间关系相比要复杂的多。
3. 绝大多数深度学习算法假设样本之间都是独立的，然而强化学习的一个sequence(序列)通常是高度相关的。
4. 当强化学习算法学习到一个新的行为时，数据服从的分布可能会改变，然而深度学习通常假设数据服从一个固定的分布。

### 方案
#### 背景
1. 智能体与环境进行交互，这里的环境是Atari模拟器。智能体不能观测到模拟器的内部状态，只能得到当前屏幕的一个数字化表示。注意目前智能体的得分取决于之间所有序列上的action和observation。一个action的feedback可能等到好几千个timesteps之后才能得到。
2. 因为智能体只能得到当前屏幕上的图像，所以这个task可以认为是部分可观测的，因为仅仅从当前的屏幕图像$x_t$上是不能完全理解当前的状况的。所有的序列都认为在有限步骤内是会结束的。
3. 智能体的目标是通过采取action和智能体交互最大化未来的reward。定义$t$时刻的回报return为$R_t = \sum^T_{t'=t}\gamma^{t'-t}r_{t'}$，其中$\gamma$是折扣因子，$T$是游戏终止的时间步。
4. 定义最优的动作值函数$Q^{\*}(s,a)$是遵循某个最优策略在状态$s$处采取动作$a$能获得的最大的期望回报，$Q^{\*(s,a)} = max_{\pi}E[R_t|s_t=s,a_t=a,\pi]$。
5. 最优的动作值函数遵循Bellman equation。如果在下个时间步的状态$s'$处，对于所有可能的$a'$，$Q^{\*}(s',a')$的最优值是已知的（这里就是对于每一个$a'$，都会有一个最优的$Q(s',a')$，最优的策略就是选择最大化$r+Q^{\*}(s',a')$的动作$a'$：
$$Q^{*}(s,a) = E_{s\sim E}[r+ \gamma max_{a'} Q^{*}(s',a')|s,a]$$
强化学习的一个思路就是使用Bellman equation更新动作值函数，$Q_{i+1}(s,a) = E[r + \gamma Q_i(s',a')|s,a]$，当$i\rightarrow \infty$时，$Q_i \rightarrow Q^{\*}$。
6. 在实践中，上述的方法是不可行的，因为动作值函数是基于每一个序列进行评估的，并没有进行泛化，当有无穷多个序列的时候，这就是不可行的，这时候可以采用函数来估计动作值函数，$Q(s,a;\theta) \approx Q^{\*}(s,a)$。一般来说，通常采用线性函数进行估计，当然可以采用非线性的函数，如神经网络等等。这里就采用神经网络，用$\theta$表示网络的参数，这里我们把这个网络叫做Q网络，Q网络可以通过最小化下列loss进行训练：
$$L_i(\theta_i) = E_{s,a\sim \rho(\cdot)}\left[(y_i - Q(s,a;\theta_i))^2\right]$$
其中$y_i = E_{s'\sim E}[r+\gamma max_{a'}Q(s',a';\theta_{i-1})]$是第$i$次迭代的target值，其中$\rho(s,a)$是$(s,a)$服从的概率分布。
7. 注意在优化$L_i(\theta_i)$时，上一次迭代的$\theta_{i-1}$是不变的，注意网络的target是取决于网络参数的，和监督学习作对比，监督学习的target和网络无关，在一开始时就是固定的。
8. 对Loss函数进行求导，得到下列的gradient信息：
$$\nabla_{\theta_i}L_i(\theta_i) = E_{s,a~\rho(\cdot),s'\sim E}\left[(r+\gamma max_{a'}Q(s',a';\theta_{i-1})-Q(s,a;\theta_i))\nabla_{\theta_i}Q(s,a;\theta_i)\right]$$
作者通过SGD优化loss函数而不是计算整个期望。如果权重是每隔几个timestep进行更新，并且用从分布$\rho$和环境$E$中采样得到的样本取代期望，就可以得到熟悉的Q-learning算法[2]。(这个具体为什么是这样，我也不清楚，可以看参考文献2)
9. 这个算法是一个Model-Free的模型，因为它直接从环境$E$中采样，并没有显式的对环境进行建模。
10. 它是一个off-policy算法，它评估策略的时候采用的是贪心策略，但是生成数据的策略和评估时是不同。生成数据的策略包含了对环境的exploration(探索)，而评估策略采用的是贪心策略，不包含探索。
> On-policy methods attempt to evaluate or improve the policy that is used to make decisions, whereas  off-policy methods evaluate or improve a policy different from that used to generate the data.

Sarsa和Q-learning的区别在于更新$Q(s,a)$时,$s'$处action采用的是原来的策略(behaviour policy（行为策略），$\epsilon$-greedy还是现在新的策略(greedy)决定，这其实就是policy evaluation和value iteration的区别，policy evaluation使用动态规划算法更新$V(s)$，但是并没有改变行为策略，更新迭代用的数据都是利用之前的行为策略生成的。而值迭代是policy evaluation+policy improvement，每一步都用贪心策略选择出最大的$a$更新$V(s)$，评估用的策略（贪心策略）和行为策略（$\epsilon$-策略）是不同的。

#### 算法
1. DQN使用了experience replay，将多个episodes中的经验存储到同一个replay buffer中。在更新$Q$值的时候，从replay buffer中进行采样更新。当前时间步动作的选择采用的是$\epsilon$-greedy策略，保持探索。因为replay buffer中存放的有很久之前的experience，所以更新$Q$值的策略(replay buffer)和真实采取动作的策略($epsilon$-greedy)是不一样的，所以是off-policy的方法。采用experience replay的online算法[5]和标准的online算法相比有三个好处[4]，第一个是每一个experience可以多次用来更新参数，提高了数据训练效率；第二个是直接从连续的样本中进行学习是低效的，因为样本之间存在强关联性。第三个是on-policy的学习中，当前的参数决定下一次采样的样本，就可能使学习出来的结果发生偏移。
2. replay buffer中只存储最近N个experience。
3. 原始图像是$210\times 160$的RGB图像，预处理首先将它变为灰度图，并进行下采样得到一个$110\times 84$的图像，然后从这个图像中截取一个$84\times 84$的图像。
4. 作者使用预处理函数$\phi$处理连续四张的图像，而不是仅仅一张，然后将这个预处理后的结果输入$Q$函数。
5. 预处理函数$\phi$是一个卷积神经网络，输入是$84\times 84\times 4$的图像矩阵，经过$16$个stride为$4$的$8\times 8$filter，经过relu激活函数，再经过$32$个stride为$2$的$4\times 4$filter，经过relu激活函数，最后接一个256个单元的全连接层。输出层的单元根据动作的个数决定。
6. $Q$函数的输入是预处理后的图像，也就是状态，输出是所有action的$Q$值，也就是说给定一个状态，这个网络能够计算所有的action的值。
7. DQN是不收敛的。

### 伪代码
Algorithm 1 Deep Q-learning with Experience Replay
Initialize replay memory D to capacity N
Initialize action-value function Q with random weights
for episode = $1, M$ do
$\ \ \ \ \ \ \ \ $Initialize sequence $s_1 = {x_1}$ and preprocessed sequenced $\phi_1 = \phi(s_1)$
$\ \ \ \ \ \ \ \ $for $t = 1,T$ do
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $With probability $\epsilon$ select a random action $a_t$ 
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $otherwise select $a_t = max_a Q^{∗}(\phi(s_t), a; θ)$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Execute action $a_t$ in emulator and observe reward $r_t$ and image $x_{t+1}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $s_{t+1} = s_t, a_t, x_{t+1}$ and preprocess $\phi_{t+1} = \phi(s_{t+1})$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Store transition $(\phi_t, a_t, r_t, \phi_{t+1})$ in D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Sample random minibatch of transitions $(\phi_j, a_j, r_j, \phi_{j+1})$ from D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $y_j = \begin{cases}r_j&\ \ \ \ for\ terminal\ \phi_{j+1}\\r_j+\gamma max_{a'}Q(\phi_{j+1},a'|\theta)&\ \ \ \ for\ non-terminal\ \phi_{j+1}\end{cases}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Perform a gradient descent step on $(y_j − Q(\phi_j, a_j; θ))^2$
$\ \ \ \ \ \ \ \ $end for
end for

### 实验
#### Datasets
七个Atari 2600 games: B.Rider, Breakout, Enduro, Pong, Q bert, Seaquest, S.Invaders。
在六个游戏上DQN的表现超过了之前所有的方法，在三个游戏上DQN的表现超过了人类。

#### Settings
1. 不同游戏的reward变化很大，这里把正的reward全部设置为$1$，把负的reward全部设置为$-1$，reward为$0$的保持不变。这样子在不同游戏中也可以统一学习率。
2. 采用RMSProp优化算法，batchsize为$32$，behaviour policy采用的是$epsilon-greedy$，在前$100$万步内，$epsilon$从$1$变到$0.1$，接下来保持不变。
3. 使用了跳帧技术，每隔$k$步，智能体才选择一个action，在中间的$k-1$步中，保持原来的action不变。这里选择了$k=4$，有的游戏设置的为$k=3$。

#### Metrics
##### average reward
第一个metric是在一个episode或者一次游戏内total reward的平均值。这个metric带有很大噪音，因为policy权值一个很小的改变可能就会对policy访问states的分布造成很大的影响。
##### action value function
第二个metric是估计的action-value function，这里作者的做法是在训练开始前使用random policy收集一个固定的states set，然后track这个set中states最大预测$Q$值的平均。

#### Baselines
1. Sarsa
2. Contingency
3. DQN
4. Human
 
### 代码
https://github.com/devsisters/DQN-tensorflow

## Nature DQN
### 非线性拟合函数不收敛的原因
1. 序列中状态的高度相关性。
2. $Q$值的一点更新就会对policy改变造成很大的影响，从而改变数据的分布。
3. 待优化的$Q$值和target value(目标Q值)之间的关系，每次优化时的目标Q值都是固定上次的参数得来的，优化目标随着优化过程一直在变。
前两个问题是通过replay buffer解决的，第三个问题是Natura DQN中解决的，在初始版本中没有解决，解决方案是在固定时间步内，固定目标Q值的参数，更新待优化Q值的参数，然后每隔固定步数将待优化Q值的参数拷贝给目标Q值。
> This instability has several causes: the correlations present in the sequence of observations, the fact that small updates to Q may significantly change the policy and therefore change the data distribution, and the correlations between the action-values (Q) and the target values $r+\gamma max_{a'}Q(s',a')$. 
> We address these instabilities with a novel variant of Q-learning, which uses two key ideas. First, we used a biologically inspired mechanism termed experience replay that randomizes over the data, thereby removing correlations in the observation sequence and smoothing over changes in the data distribution. Second, we used an iterative update that adjusts the action-values (Q) towards target values that are only periodically updated, thereby reducing correlations with the target.

### Nature DQN改进 
1. 预处理的结构变了,CNN的层数增加了一层，
2. 加了target network，
3. 将error限制在$[-1,1]$之间。
> clip the error term from the update $r + \gamma max_{a'} Q(s',a';\theta_i^{-} - Q(s,a;\theta_i)$ to be between $-1$ and $1$. Because the absolute value loss function $|x|$ has a derivative of $-1$ for all negative values of $x$ and a derivative of $1$ for all positive values of $x$, clipping the squared error to be between $-1$ and $1$ corresponds to using an absolute value loss function for errors outside of the $(-1,1)$ interval. 

### 框架
DNQ的框架如下所示
![ndqn](nature_dqn.png)

### 伪代码
Algorithm 2 deep Q-learning with experience replay, target network
Initialize replay memory D to capacity N
Initialize action-value function Q with random weights $\theta$
Initialize target action-value function $\hat{Q}$ with weights $\theta^{-}=\theta$
for episode = $1, M$ do
$\ \ \ \ \ \ \ \ $Initialize sequence $s_1 = {x_1}$ and preprocessed sequenced $\phi_1 = \phi(s_1)$
$\ \ \ \ \ \ \ \ $for $t = 1,T$ do
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $With probability $\epsilon$ select a random action $a_t$ 
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $otherwise select $a_t = max_a Q^{∗}(\phi(s_t), a; θ)$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Execute action $a_t$ in emulator and observe reward $r_t$ and image $x_{t+1}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $s_{t+1} = s_t, a_t, x_{t+1}$ and preprocess $\phi_{t+1} = \phi(s_{t+1})$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Store transition $(\phi_t, a_t, r_t, \phi_{t+1})$ in D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Sample random minibatch of transitions $(\phi_j, a_j, r_j, \phi_{j+1})$ from D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $y_j = \begin{cases}r_j&\ \ \ \ for\ terminal\ \phi_{j+1}\\r_j+\gamma max_{a'}Q(\phi_{j+1},a'|\theta^{-})&\ \ \ \ for\ non-terminal\ \phi_{j+1}\end{cases}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Perform a gradient descent step on $(y_j − Q(\phi_j, a_j; θ))^2$ with respect to the network parameters $\theta$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Every $C$ steps reset $\hat{Q} = Q$
$\ \ \ \ \ \ \ \ $end for
end for

## Double DQN
## Prioritized DDQN
## Dueling DQN
## Distributed DQN
## Noisy DQN
## Rainbow

## 参考文献
1.https://blog.csdn.net/yangshaokangrushi/article/details/79774031
2.https://link.springer.com/article/10.1007%2FBF00992698
3.https://www.jianshu.com/p/b92dac7a4225
4.https://datascience.stackexchange.com/questions/20535/what-is-experience-replay-and-what-are-its-benefits/20542#20542
5.https://stats.stackexchange.com/questions/897/online-vs-offline-learning
