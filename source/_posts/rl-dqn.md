---
title: DQN
date: 2019-03-02 19:29:35
tags: 
 - 强化学习
categories: 强化学习
mathjax: true
---

## 背景
1. Atari 2600是一个RL benchmark，有2600个游戏，每个agent会得到一个图像输入(60Hz的210 x 160 RGB视频)。本文的目标是设计一个NN架构尽可能学会更多游戏，网络的输入只有视频信息，reward和terminal信号以及可能采取的action，和人类玩游戏时得到的信息是一样的。
2. Agent与Atari模拟器不断交互，agent不能观测到模拟器的内部状态，只能得到当前屏幕信息的一个图片。这个task可以认为是部分可观测的，因为仅仅从当前的屏幕图像$x_t$上是不能完全理解整个游戏状况的。所有的序列都认为在有限步骤内是会结束的。
3. 注意agent当前的得分取决于整个sequence的action和observation。一个action的feedback可能等到好几千个timesteps之后才能得到。
4. agent的目标最大化累计reward。定义$t$时刻的回报return为$R_t = \sum\^T\_{t'=t} \gamma\^{t'-t}r\_{t'}$，其中$\gamma$是折扣因子，$T$是游戏终止的时间步。
5. 定义最优的动作值函数$Q\^{\*}(s,a)$是遵循最优策略在状态$s$处采取动作$a$能获得的最大的期望回报，$Q\^{\*}(s,a) = \max\_{\pi}E[R_t|s_t=s,a_t=a,\pi]$。
6. 最优的动作值函数遵循Bellman optimal equation。如果在下个时间步的状态$s'$处，对于所有可能的$a'$，$Q\^{\*}(s',a')$的最优值是已知的（这里就是对于每一个$a'$，都会有一个最优的$Q(s',a')$，最优的策略就是选择最大化$r+Q\^{\*}(s',a')$的动作$a'$：
$$Q\^{\*}(s,a) = E_{s\sim E}[r+ \gamma \max_{a'} Q\^{\*}(s',a')|s,a], \tag{1}$$
强化学习的一个思路就是使用Bellman optimal equation更新动作值函数，$Q_{i+1}(s,a) = E[r + \gamma Q_i(s',a')|s,a]$，当$i\rightarrow \infty$时，$Q_i \rightarrow Q\^{\*}$。
7. 上述例子是state-action pair很少的情况，当有无穷多个的时候，是无法精确计算的。这时候可以采用函数来估计动作值函数，$Q(s,a;\theta) \approx Q\^{\*}(s,a)$。一般来说，通常采用线性函数进行估计，当然可以采用非线性的函数，如神经网络等等。这里采用的是神经网络，用$\theta$表示网络的参数，这个网络叫做Q网络，Q网络通过最小化下列loss进行训练：
$$L_i(\theta_i) = E_{s,a\sim \rho(\cdot)}\left[(y_i - Q(s,a;\theta_i))\^2\right]\tag{2}$$
其中$y_i = E_{s'\sim E}[r+\gamma \max_{a'}Q(s',a';\theta\_{i-1})]$是第$i$次迭代的target值，其中$\rho(s,a)$是$(s,a)$服从的概率分布。
8. 注意在优化$L_i(\theta_i)$时，上一次迭代的$\theta\_{i-1}$是不变的，target取决于网络参数，和监督学习作对比，监督学习的target和网络参数无关。
9. 对Loss函数进行求导，得到下列的gradient信息：
$$\nabla_{\theta_i}L_i(\theta_i) = E_{s,a\sim \rho(\cdot),s'\sim E}\left[(r+\gamma \max_{a'}Q(s',a';\theta_{i-1})-Q(s,a;\theta_i))\nabla_{\theta_i}Q(s,a;\theta_i)\right]\tag{3}$$
通过SGD优化loss函数。如果权重是每隔几个timestep进行更新，并且用从分布$\rho$和环境$E$中采样得到的样本取代期望，就可以得到熟悉的Q-learning算法[2]。(这个具体为什么是这样，我也不清楚，可以看参考文献2)
10. 什么是on-polciy算法： 
> On-policy methods attempt to evaluate or improve the policy that is used to make decisions, whereas  off-policy methods evaluate or improve a policy different from that used to generate the data.

Sarsa和Q-learning的区别在于更新Q值时的target policy和behaviour policy是否相同。我觉的是是policy evaluation和value iteration的区别，policy evaluation使用动态规划算法更新$V(s)$，但是并没有改变行为策略，更新迭代用的数据都是利用之前的行为策略生成的。而值迭代是policy evaluation+policy improvement，每一步都用贪心策略选择出最大的$a$更新$V(s)$，target policy（greedy）和behaviour policy（$\varepsilon$-greedy）是不同的。

## 强化学习需要解决的问题
1. 大量有标记的训练数据。
2. delayed-reward。这个delay存在于action和reward之间，可以达到几千个timesteps那么远，和supervised learnign中输入和输入之间直接的关系相比要复杂的多。
3. 大多数深度学习算法假设样本之间都是独立的，然而强化学习的一个sequence(序列)通常是高度相关的。
4. 强化学习算法学习到的policy变化时，数据服从的分布通常会改变，然而深度学习通常假设数据服从一个固定的分布。

## DQN
论文名称[Playing Atari with Deep Reinforcement Learning](https://arxiv.org/pdf/1312.5602.pdf)
### 概述
DQN算法使用卷积神经网络代替Q-learning中tabular的值函数，并提出了几个trick促进收敛。DQN agnet的输入是原始的图片，输出是图片表示的state可能采取的action的$Q$值。
1. dqn是Model-Free的，它直接从环境$E$中采样，并没有显式的对环境进行建模。
2. dqn是一个online的方法，即训练数据不断增加；offline是训练数据固定。
3. dqn是一个off-policy算法，target policy 是greedy policy，behaviour policy是$\varepsilon$-greedy policy，target policy和greedy policy策略不同。
4. DQN是不收敛的。


### 解决方案
#### Experience replay
1. DQN使用了experience replay，将多个episodes中的经验存储到一个大小为$N$的replay buffer中。在更新$Q$值的时候，从replay buffer中进行采样更新。behaviour policy是$\varepsilon$-greedy策略，保持探索。target policy是$\varepsilon$ greedy 算法，因为replay buffer中存放的都是behaviour policy生成的experience，所以是off-policy算法。
采用experience replay的DQN和Q-learning算法相比有三个好处，第一个是每一个experience可以多次用来更新参数，提高了数据训练效率；第二个是直接从连续的样本中进行学习是低效的，因为样本之间存在强关联性。第三个是on-policy的学习中，当前的参数决定下一次采样的样本，就可能使学习出来的结果发生偏移。
2. replay buffer中只存储最近N个experience。

#### Data preprocess
1. 原始图像是$210\times 160$的RGB图像，预处理首先将它变为灰度图，并进行下采样得到一个$110\times 84$的图像，然后从这个图像中截取一个$84\times 84$的图像。
2. 作者使用预处理函数$\phi$处理连续四张的图像而不是一张，然后将这个预处理后的结果输入$Q$函数。
3. 预处理函数$\phi$是一个卷积神经网络，输入是$84\times 84\times 4$的图像矩阵，经过$16$个stride为$4$的$8\times 8$filter，经过relu激活函数，再经过$32$个stride为$2$的$4\times 4$filter，经过relu激活函数，最后接一个256个单元的全连接层。输出层的大小根据不同游戏的动作个数决定。
4. $Q$网络的输入是预处理后的图像state，输出是所有当前state可能采取的action的$Q$值。

### 网络结构
输入：[batch_size, 84, 84, 4]
第一个隐藏层：16个步长为$4$的$8\times 8$的filters
第二个隐藏层：32个步长为$2$的$4\times 4$的filters
全连接层：256个units
输出层：softmax

### 算法
算法 1 Deep Q-learning with Experience Replay
Initialize replay memory D to capacity N
Initialize action-value function Q with random weights
for episode = $1, M$ do
$\ \ \ \ \ \ \ \ $Initialize sequence $s_1 = {x_1}$ and preprocessed sequenced $\phi_1 = \phi(s_1)$
$\ \ \ \ \ \ \ \ $for $t = 1,T$ do
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $With probability $\varepsilon$ select a random action $a_t$ 
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $otherwise select $a_t = \max_a Q\^{∗}(\phi(s_t), a; θ)$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Execute action $a_t$ in emulator and observe reward $r_t$ and image $x_{t+1}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $s_{t+1} = s_t, a_t, x_{t+1}$ and preprocess $\phi_{t+1} = \phi(s_{t+1})$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Store transition $(\phi_t, a_t, r_t, \phi_{t+1})$ in D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Sample random minibatch of transitions $(\phi_j, a_j, r_j, \phi_{j+1})$ from D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $y_j = \begin{cases}r_j&\ \ \ \ for\ terminal\ \phi_{j+1}\\\\r_j+\gamma \max_{a'}Q(\phi_{j+1},a'|\theta)&\ \ \ \ for\ non-terminal\ \phi_{j+1}\end{cases}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Perform a gradient descent step on $(y_j − Q(\phi_j, a_j|θ))\^2$
$\ \ \ \ \ \ \ \ $end for
end for

### Experiments
#### Datasets
七个Atari 2600 games: B.Rider, Breakout, Enduro, Pong, Q bert, Seaquest, S.Invaders。
在六个游戏上DQN是SOTA，在三个游戏上DQN的表现超过了人类。

#### Settings
1. 不同游戏的reward变化很大，这里把正的reward全部设置为$1$，把负的reward全部设置为$-1$，reward为$0$的保持不变。这样子在不同游戏中也可以统一学习率。
2. 采用RMSProp优化算法，batch size为$32$，behaviour policy采用的是$\varepsilon$-greedy，在前$100$万步内，$\varepsilon$从$1$变到$0.1$，接下来保持不变。
3. 使用了fram-skip技术，每隔$k$步，agent才选择一个action，在中间的$k-1$步中，保持原来的action不变。这里选择了$k=4$，有的游戏设置的为$k=3$。
4. 超参数设置没有说

#### Metrics
每个agent训练$10$ millions帧，replay buffer size是$1$ million。每个epoch进行$50000$个minibatch weight updates或者大约$30$分钟的训练（这里有些不理解）。然后使用$\epsilon$-greedy($\epsilon=0.05$) evaluation $10000$个steps。

##### average total reward
第一个metric是在一个episode或者一次游戏内total reward的平均值。这个metric带有很大噪音，因为policy权值一个很小的改变可能就会对policy访问states的分布造成很大的影响。

##### action value function
第二个metric是估计的action-value function，这里作者的做法是在训练开始前使用random policy收集一个固定的states set，然后track这个set中states最大预测$Q$值的平均。尽管缺乏理论收敛保证，DQN看起来还不错。

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
前两个问题是通过DQN中提出的replay buffer解决的，第三个问题是Natura DQN中解决的，在一定时间步内，固定target network参数，更新待network的参数，然后每隔固定步数将network的参数拷贝给target network。
> This instability has several causes: the correlations present in the sequence of observations, the fact that small updates to Q may significantly change the policy and therefore change the data distribution, and the correlations between the action-values (Q) and the target values $r+\gamma \max_{a'}Q(s',a')$. 
> We address these instabilities with a novel variant of Q-learning, which uses two key ideas. First, we used a biologically inspired mechanism termed experience replay that randomizes over the data, thereby removing correlations in the observation sequence and smoothing over changes in the data distribution. Second, we used an iterative update that adjusts the action-values (Q) towards target values that are only periodically updated, thereby reducing correlations with the target.

### 解决方案
1. 预处理的结构变了,CNN的层数增加了一层，
2. 加了target network，
3. 将error限制在$[-1,1]$之间。
> clip the error term from the update $r + \gamma \max_{a'} Q(s',a';\theta_i\^{-} - Q(s,a;\theta_i)$ to be between $-1$ and $1$. Because the absolute value loss function $|x|$ has a derivative of $-1$ for all negative values of $x$ and a derivative of $1$ for all positive values of $x$, clipping the squared error to be between $-1$ and $1$ corresponds to using an absolute value loss function for errors outside of the $(-1,1)$ interval. 

### 框架和网络结构
#### 框架
Nature-DNQ的框架如下所示
![ndqn](nature-dqn.png)

#### 网络结构
输入:[batch_size, 84, 84, 4]
三个卷积层，两个全连接层（包含输出层）
第一个隐藏层：$32$个步长为$4$的$8\times 8$filters，以及一个relu
第二个隐藏层：$64$个步长为$2$的$4\times 4$filters，以及一个relu
第三个隐藏层：$64$个步长为$1$的$3\times 3$filters，以及一个relu
第四个隐藏层：$512$个units
输出层：softmax，输出每个action对应的$Q$值

### 算法
算法 2 deep Q-learning with experience replay, target network
Initialize replay memory D to capacity N
Initialize action-value function Q with random weights $\theta$
Initialize target action-value function $\hat{Q}$ with weights $\theta\^{-}=\theta$
for episode = $1, M$ do
$\ \ \ \ \ \ \ \ $Initialize sequence $s_1 = {x_1}$ and preprocessed sequenced $\phi_1 = \phi(s_1)$
$\ \ \ \ \ \ \ \ $for $t = 1,T$ do
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $With probability $\varepsilon$ select a random action $a_t$ 
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $otherwise select $a_t = \max_a Q\^{∗}(\phi(s_t), a; θ)$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Execute action $a_t$ in emulator and observe reward $r_t$ and image $x_{t+1}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $s_{t+1} = s_t, a_t, x_{t+1}$ and preprocess $\phi_{t+1} = \phi(s_{t+1})$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Store transition $(\phi_t, a_t, r_t, \phi_{t+1})$ in D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Sample random minibatch of transitions $(\phi_j, a_j, r_j, \phi_{j+1})$ from D
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Set $y_j = \begin{cases}r_j&\ \ \ \ for\ terminal\ \phi_{j+1}\\\\r_j+\gamma \max_{a'}Q(\phi_{j+1},a'|\theta\^{-})&\ \ \ \ for\ non-terminal\ \phi_{j+1}\end{cases}$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Perform a gradient descent step on $(y_j − Q(\phi_j, a_j|θ))\^2$ with respect to the network parameters $\theta$
$\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ $Every $C$ steps reset $\hat{Q} = Q$
$\ \ \ \ \ \ \ \ $end for
end for

### Experiments
#### Settings
- batch-size: 32
- replacy memory size: 1000000 frames
- target network update frequency: 10000
- discount factor: 0.99
- action repeat: 4 # 就是frame skip
- history length: 4 # 使用最近的几帧重叠，实际上是16帧
- paramteter update frequency: 4 # 执行sgd train的frequency
- learning rate: 0.00025
- gradient momentum: 0.95
- squared gradient momentum: 0.95
- min squared gradient: 0.01
- initial exploration: 1
- final exploration 0.1
- final exploration frame: 1000000
- replay start size: 50000
- no-op max: 30
- reward: clipped to [-1, 1]
- total train frames: 50 millons frame（实际上是200 millions emulated frames，因为有设置为$4$ frame skip）。
- 选择random作为一个baseline，因为人类的极限是$10$hz，为了公平起见，random baseline以$10$hz的频率随机选择一个action，atari视频的频率是$60$hz，所以每隔$6$帧，随机选择一个action，在选择action中间的帧中保持这一个action。

#### Experiments
- Average score和average action value
每个training epoch之后进行一次evaluation，记录evaluation过程中average episode reward。总共train $50$million frames，大概有$200$个epoch，也就是一个epoch是$25$万frames，在每个epoch后使用$\epsilon$-greedy($\epsilon =0.05$)策略evaluate $520k$个frames。
- Main-Evaluation: Compartion between DQN and other baselines
Baselines有Random play, best linear learner，SARSA，Huamn等。
DQN总共训练了$50$ million frames，replay buffer存放最近的$1$ million framems。在完成训练后，至多执行$30$次no-op，产生随机初始状态，使用$\epsilon$-greedy($\epsilon=0.05$)玩$5$分钟，对多次结果取平均。
Human的数据是在玩家首先进行了$2$个小时训练后，然后玩大约20 episodes，每个episode最长5 min的 average reward。
表格中最后一列还给出了一个百分比，$100\times \frac{\text{DQN score} - \text{random play score}}{\text{human score} - \text{random play score}}$。
- Replay buffer和target network的abalation实验
在三个不同的learning rate下使用standard hyperparameters训练DQN $10$ million frames。每隔$250,000$ training frames对每个agent进行$135,000$ frames的validation，记录最高的average episode score。 这些valuation episodes并没有在$5$ min的时候截断，这个实验中Enduro得到的score要比main evaluation中高。这个实验中training的帧数($10$ million frames)要比baseline中training的frames($50$ million frames)少。
- DQN和linear function approximator比较
除了function approximator由CNN变成linear的，其他都没有变。
在$5$个validation games上，每个agent在三个不同的learning rates使用标准的参数训练了$10$ million frames。每隔$250000$个training frames对agent进行$135,000$ frames的validation，reported 最高的average episode score。 这些valuation episodes并没有在$5$ min的时候截断，这个实验中Enduro得到的score要比main evaluation中高。这个实验中training的帧数($10$ million frames)要比baseline中training的frames($50$ million frames)少。

## Gorila DQN
论文名称：
Massively Parallel Methods for Deep Reinforcement Learning
下载地址：
https://arxiv.org/pdf/1507.04296.pdf

### Experiments

#### Settings
- 网络结构和nature DQN一样。
- 使用了frame-skip，设置为$4$。
- Replay memory是$1$M>

#### Evaluation
Evaluation有两种：

##### null op starts
每个agent在它训练的游戏上evaluated $30$个episodes，每个episode随机的至多执行$30$次no-op之后，评估$5$min的emulator时间($18000$ frames)。然后取这$100$次的平均值。

##### human starts
Human starts是用来衡量它对于agent可能没有遇到过的state的泛化能力。对于每一个游戏，从一个人类玩家的gameplay中随机取$100$个开始点，使用$\epsilon$-greedy policy玩三十分钟emulator时间（即$108000$frames)。

## Double DQN
### DQN中的overestimate问题
解决overestimate问题，Q-learning中在estimated values上进行了max操作，可能会导致某些更偏爱overestimated value而不是underestimated values。
本文将Double Q-learning的想法推广到了dqn上形成了double-dqn。实验结果表明了overestimated value对于policy有影响，double 会产生好的action value，同时在一些游戏上会得到更高的scores。

### Contributions
1. 解释了在large scale 问题上，Q-learning被overoptimistic的原因是学习固有的estimation errors。
2. overestimation在实践中是很常见，也很严重的。
3. Double Q-learning可以减少overoptimism
4. 提出了double-dqn。
5. double-dqn在某些游戏上可以找到更好的policy。

### Double Q-learning
Q-learning算法计算target value $y$的公式如下：
$$y = r + \gamma \max_a' Q(s', a'|\theta_t)\tag{4}$$
在计算target value的时候，使用同一个网络选择和评估action $a'$，这可能会让网络选择一个overestimated value，最后得到一个overoptimistic value estimate。所有就有了double Q-learning，计算公式如下：
$$y = r + \gamma Q(s', \arg\max_a' Q(s',a;\theta_t);\theta'\_t)\tag{5}$$
target policy还是greedy policy，通过使用$\theta$对应的网络选择action，然后在计算target value的时候使用$\theta'$对应的网络。
原有的公式可以写成下式，
$$y = r + \gamma Q(s', \arg\max_a' Q(s',a;\theta_t);\theta_t)\tag{6}$$
即选择action和计算target value都是使用的同一个网络。

### Double DQN
![double-dqn](double-dqn.png)
Double Q-learnign的做法是分解target action中的max opearation为选择和evaluation。而在Nature-dqn中，提出了target network，所以分别使用network和target network去选择和evaluation action是一个很好的做法，这样子公式就变成了
$$y = r + \gamma Q(s', \arg\max_a' Q(s',a;\theta_t);\theta\^{-}\_t)\tag{7}$$
和Q-learnign相比，将$\theta'$换成了$\theta\^{-}$ evaluate action，target network的更新和nature-dqn一样，过一段时间复制network的参数。

### Double Q learning vs Q-learning
可以在数学上证明，Q-learning是overestimation的，但是double q leraing是无偏的。。。证明留待以后再说。
<TODO>

### 网络结构
网络结构和nature DQN一样。

### 算法
算法 3: Double DQN Algorithm.
输入: replay buffer $D$, 初始network参数$\theta$,target network参数$\theta\^{-}$ 
输入 : replay buffer的大小$N_r$, batch size $N_b$, target network更新频率$N\^{-}$
**for** episode $e \in \{1, 2,\cdots, M\}$ do
$\qquad$初始化frame sequence $\mathbf{x} \leftarrow ()$
$\qquad$**for** $t \in \{0, 1, \cdots\}$ do
$\qquad\qquad$设置state $s \leftarrow \mathbf{x}$, 采样 action $a \sim\pi_B$
$\qquad\qquad$给定$(s, a)$，从环境$E$中采样接下来的frame $x_t$,接收reward $r$,在序列$\mathbf{x}$上拼接$x$
$\qquad\qquad$**if** $|\mathbf{x}| \gt N_f$
$\qquad\qquad$**then** 
$\qquad\qquad\qquad$从$\mathbf{x}$中删除最老的frame $x\_{t_min}$
$\qquad\qquad$设置$s' \leftarrow \mathbf{x}$,添加transition tuple (s, a, r, s 0 ) 到buffer D中，如果$|D| \ge N_r$替换最老的tuple 
$\qquad\qquad$采样$N_b$个tuples $(s, a, r, s') \sim Unif(D)$
$\qquad\qquad$计算target values, one for each of $N_b$ tuples:
$\qquad\qquad$定义$a\^{\max}(s'; \theta) = \arg \max\_{a'} Q(s', a';\theta)$
$\qquad\qquad y_j = \begin{cases}r&\qquad if\ \ s'\ \ is\ \ terminal\\\\ r+\gamma Q(s', a\^{\max}(s';\theta);\theta\^{-}, &\qquad otherwise\end{cases}$
$\qquad\qquad$利用loss $||y_j − Q(s, a; \theta)||^2$的梯度更新
$\qquad\qquad$每隔$N\^{-}$个步骤更新一下target network 参数$\theta\^{-}$
$\qquad$**end**
**end**

### Experiments

#### Settings
Tunned Double DQN，update frequency从$10000$改成了$30000$，训练时$\epsilon$在$1$ millon内从$0.1$退火到$0.01$。Evaluation时是$0.001$。

#### Evaluation
和Gorila DQN一样，用了两种：no-op和human starts。

#### Training
在每个游戏上，网络都训练了$200$M frames，也就是$50$M steps。每隔$1$M step进行一次evaluation，从evaluations中选出最好的policy作为输出。

#### Metric
提出了一个指标，normalized score，计算公式如下：
$$score\_{normalized} = \frac{score\_{agent}- score\_{random}}{score\_{human}-score\_{random}}\tag{8}$$
分母是human和random之差，对应$100%$。


## Prioritized DQN(PER)
### contributions
本文提出一种了proritizing experience的框架，在训练过程中多次使用重要的transtions replay进行更新，让训练变得的更有效率。
使用TD-errors作为prioritization mechanism，给出了两种protitization计算方式，提出了一种stochastic prioritization以及importance sampling方法。

### Prioritized replay
可以从两个维度上考虑replay memeory的改进，一个是存哪些experiences，一个是使用哪些experiences进行回放。本文是从第二个方向上进行的考虑。

从buffer中随机抽样的方法中，update steps和memory size是线性关系，作者想找一个update steps和memory size是log关系的oracle，但是很遗憾，这是不现实的，所以作者想要找一种比uniform random replay好尽量接近oracle的方法。

#### Prioritizion with TD-error
prioritized replay最重要的部分是如何评价每一个transition的重要程度。一个理想的criterion是agent在当前的state可以从某个transition中学到多少。这个measure metric是不确定的，一个替代方案是使用TD error $\delta$，表示how 'suprising' 或者upexpected the transition：就是当前的value离next-step bootstrap得到的value相差多少，booststrap就是基于其他估计值进行计算。。这中方法对于incremental,online RL方法，例如SARSA以及Q-learning来说都是很合适的，因为他们会计算TD-error，然后给TD-error一个比例系数用来更新参数。然后当reward是noisy的时候，TD-error效果可能很差。
作者在一个人工设计的环境中使用了greedy TD-error prioritization算法，算法在每次存transition到replay buffer的时候，同时还会存一下该transition最新的TD-error，然后在更新的时候从memory中选择TD-error最大的transition。最新的transition TD-error没有算出来，就给它一个最大的priority，保证所有的experience都至少被看到过一次。
采用二叉堆用实现优先队列，查找复杂度是$O(1)$，更新priorities的复杂度是$O(logN)$。

#### Stochastic prioritization
上述方法有很多问题。第一，每次都sweep整个replay memory的计算量很大，所以只有被replayed的experiences的TD-errors才会被更新。开始时一个TD error很小的transition可能很长一段事件不会被replayed，这就导致了replay buffer的sliding window不起作用了。第二，TD-error对于noise spike很敏感，还会被bootstrap加剧，估计误差可能会是另一个noise。第三，greedy prioritization集中在experiences的一个subset：errors减小的很慢，尤其是使用function appriximation时，这就意味着初始的高error的transitions会被replayed的很频繁，然后会over-fitting因为缺乏diversity。
为了解决这些问题，引入了一个介于pure greedy prioritizaiton以及uniform random sampling之间的stochastic采样方法，priority高的transition有更大的概率被采样，而lowest-priority的transition也有概率被选中，具体的定义transition $i$的概率如下：
$$P(i) = \frac{p_i\^{\alpha}}{\sum_kp_k\^{\alpha}}\tag{9}$$
$\alpha$确定prioritizaiton的比重，如果$\alpha=0$就是unifrom。

有两种$p_i$的计算方法，一种是直接的proportional prioritization，$p_i = |\delta_i| + \varepsilon$，其中$\varepsilon$是一个小的正整数，确定当$p_i=0$时，该transition仍能被replay；第二种是间接的，$p_i = \frac{1}{rank(i)}$，其中$rank(i)$是所有replay memory中的experiences根据$|\delta_i|$排序后的rank。第二种方法的鲁棒性更好。
在实现上，两种方法都有相应的trick，让复杂度不依赖于memory 大小$N$。Proportional prioritization采用了'sum-tree'数据结构，每一个节点都是它的子节点的children，priorities是leaf nodes。而rank-based方法，使用线性函数估计累计密度函数，具体怎么实现没有细看。

#### annealing the bias
因为random sample方法，samples之间没有一点联系，选择每一个sample的概率都是相等的，但是如果加上了priority，就有一个bias toward高priority的samples。IS和prioritized replay的组合在non-learn FA中有一个用处，large steps可能会产生不好的影响，因为梯度信息可能是局部reliable，所以需要使用一个小点的step-size。
在本文中，high-error的样本可能会观测到很多次，使用IS减小gradient的大小，对应于高priority的samples的weight被微调了一下，而对应于低priority的样本基本不变。
weigth的计算公式如下：
$$w_i = (\frac{1}{N}\cdot \frac{1}{P(i)})\^{\beta}\tag{10}$$
OK,这里IS的作用有些不明白。。。。<TODO>

### 算法
算法 4
输入: minibatch $k$, 学习率（步长）$\eta$, replay period $K$ and size $N$ , exponents $\alpha$ and $\beta$, budget $T$.
初始化replay memory $H = \emptyset, \Delta = 0, p_1 = 1$
根据$S_0$选择 $A_0 \sim \pi\_{\theta}|(S_0)$
**for** $t = 1,\cdots, T$ do
$\qquad$观测$S_t, R_t, \gamma_t$
$\qquad$存储transition $(S\_{t−1}, A\_{t−1}, R_t , \gamma_t, S_t)$ 到replay memory，以及$p_t$的最大priority $p_t = \max {i\lt t} p_i$
$\qquad$**if** $t ≡ 0$ mod $K$ then
$\qquad\qquad$**for** j = 1 to k do
$\qquad\qquad\qquad$Sample transition $j \sim P(j) = \frac{p_j^{\alpha}}{\sum_i p_i^{\alpha}}$
$\qquad\qquad\qquad$计算importance-sampling weight $w_j = \frac{(N \cdot P(j))\^{\beta}}{\max_i w_i}$
$\qquad\qquad\qquad$计算TD-error $\delta_j = R_j + \gamma_j Q\_{target} (S_j$, $arg \max_a Q(S_j, a)) − Q(S\_{j−1} , A\ {j−1})$
$\qquad\qquad\qquad$更新transition的priority $p_j \leftarrow |\delta_j|$
$\qquad\qquad\qquad$累计weight-change $\Delta \leftarrow \Delta + w_j \cdot \delta_j \cdot \nabla\_{\theta} Q(S\_{j−1}, A\_{j−1})$
$\qquad\qquad$**end for**
$\qquad\qquad$更新weights $\theta\leftarrow \theta+ \eta\cdot\Delta$, 重置$\Delta = 0$
$\qquad\qquad$每隔一段时间更新target network $\theta\_{target} \leftarrow \theta$
$\qquad$**end if**
$\qquad$选择action $A_t \sim \pi\_{\theta}(S_t)$
**end for**

### Experiments
两组实验，
一组是DQN和proportional prioritization作比较。
一组是tuned Double DQN和rank-based以及proportional prioritizaiton。

#### Metrics
用的是double dqn提出来的nomalized score，这里在分母上加了绝对值。
主要用的median scores和mean scores。


## Dueling DQN
### 介绍
本文作者提出来将dueling网络框架应用在model-free算法上。The dueling architecture能用一个deep model同时表示$V(s)$和优势函数$A(s,a)$，网络的输出将$V$和$A$结合产生$Q(s,a)$。和advantage不一样的是，这种方式在构建时就将他们进行了解耦，因此，dueling architecture可以应用在各种各样的model free RL算法上。
本文的架构是对算法创新的补充，它可以对之前已有的各种DQN算法进行结合。

### dueling network architecture
这个新的architecture的核心想法是，没有必要估计所有states的action value。在一些states，需要action value去确定执行哪个action，但是在许多其他states，action values并没有什么用。当然，对于bootstrap算法来说，每一个state的value estimation都很重要。
![dueling-dqn](deuling-dqn.png)
作者给出了一个single Q-network的architecture，如图所示。
网络结构和nature-dqn一样，但是这里加了两个fully connected layers，一个用于输出$V$，一个用于输出$A$。然后$A$和$V$结合在一起，产生$Q$，网络的输出和nature dqn一样，对应于某个state的一系列action value。
从$Q$函数的定义$Q\^{\pi}(s,a) = V\^{\pi}(s)+A\^{\pi}(s,a)$以及$Q$和$V$之间的关系$V\^{\pi}(s) = \mathbb{E}\_{a\sim\pi(s)}\left[Q\^{\pi}(s,a)\right] = \pi(a|s)Q\^{\pi}(s,a)$，所以有$\mathbb{E}\_{a\sim\pi(s)}\left[A\^{\pi}(s,a)\right]=0$。此外，对于deterministic policy，$a\^{\*} = \arg \max\_{a'\in A}Q(s,a')$，有$V(s) = Q(s,a\^{\*})$，即$A(s,a\^{\*}) = 0$。
如图所示的network中，一个网络输出scalar $V(s;\theta, \beta)$，一个网络输出一个$|A|$维的vector $A(s,a;\theta, \alpha)$，其中$\theta$是网络参数，$\alpha$和$\beta$是两个全连接层的参数。
根据advantage的定义，可以直接将他们加起来，即：
$$Q(s,a;\theta, \alpha, \beta) = V(s;\theta, \beta) + A(s,a;\theta, \alpha) \tag{11}$$
但是，我们需要知道的一点是，$Q(s, a;\theta, \alpha, \beta)$仅仅是$Q$的一个参数化估计。它由两部分组成，一部分是$V$，一部分是$A$，但是需要注意的是，这里的$V$和$Q$只是我们叫它$V$和$A$，它的实际意义并不是$V$和$A$。给了$Q$，我们可以得到任意的$Q(s, a) = V(s) + A(s,a)$，而$V$和$Q$并不代表value function和advantage functino。
为了解决这个问题，作者提出了选择让advantage为$0$的action，即：
$$Q(s, a; \theta,\alpha, \beta) = V(s; \theta, \beta) + \left(A(s,a;\theta,\alpha) - \max\_{a'\in |A|}A(s, a'; \theta, \alpha)\right)\tag{12}$$
选择$a\^{\*} = \arg \max\_{a'\in A} Q(s, a'; \theta, \alpha, \beta) = \arg \max\_{a'\in A}A(s, a';\theta, \alpha)$，我们得到$Q(s,a\^{\*}; \theta, \alpha,\beta) = V(s;\theta, \beta)$。这个时候，输出$V$的网络给出的真的是state value的估计$V(s;\theta, \beta)$，另一个网络真的给出的是advantage的估计。
另一种方法是用mean取代max操作：
$$Q(s, a; \theta,\alpha, \beta) = V(s; \theta, \beta) + \left(A(s,a;\theta,\alpha)- \frac{1}{|A|}\sum\_{a'}A(s, a'; \theta, \alpha)\right)\tag{13}$$
一方面这种方法失去了$V$和$A$的原始语义，因为它们有一个常数的off-target，但是另一方面它增加了优化的稳定性，因为上式中advantage的改变只需要和mean保持一致即可，不需要optimal action's advantange一有变化就要改变。

### 算法

## Distributed DQN


## Noisy DQN
### 介绍
已有方法的exploration都是通过agent policy的random perturbations，比如常见的$\varepsilon$-greedy等方法。这些方法不能找出环境中efficient exploration的behavioural patterns。常见的方法有以下几种:
第一种方法是optimism in the face of uncertainty，理论上证明可行，但是通常应用在state-action spaces很小的情况下或者linear FA，很难处理non-linearn FA，而且non-linear情况下收敛性没有保证。
另一种方法是添加额外的intrinsic motivation term，该方法的问题是将算法的generalisation mechanism和exploration分割开，即有instrinsic reward和environment reward，它们的比例如何去设置，需要认为指定。如果不仔细调整，optimal policy可能会受intrinsic reward影响很大。此外为了增加exploration的鲁邦性，扰动项仍然是需要的。这些算法很具体也能应用在参数化policy上，但是很低效，而且需要很多次policy evaluation。
本文提出NoisyNet学习网络参数的perturbations，主要想法是参数的一点改变可能会导致policy在很多个timsteps上的consistent，complex, state-dependent的变化，而如$\varepsilon$-greedy的dithering算法中，每一步添加到policy上的noise都是不相关的。pertubations从一个noise分布中进行采样，它的variance可以看成noise的energy，variance的参数和网络参数都是通过loss的梯度进行更新。网络参数中仅仅加入了噪音，没有distribution，可以自动学习。
在高维度上，本文的算法是一个randomised value function，这个函数是neural network，网络的参数并没有加倍，linear 的参数加倍，而参数是noise的一个简单变换。
还有人添加constant Gaussian niose到网络参数，而文本的算法添加的noise并不是限制在Gaussion noise distributions。添加noise辅助训练在监督学习等任务中一直都有，但是这些噪音都是不能训练的，而NoisyNet中的噪音是可以梯度下降更新的。

### NoisyNets 
![noisy_linear_layer](noisy_linear_layer.png)
用$\theta$表示noisy net的参数，输入是$x$，输出是$y$，即$y=f\_{\theta}(x)$。$\theta$定义为$\theta=\mu+\Sigma\odot\varepsilon$，其中$\zeta=(\mu,\Sigma)$表示可以学习的参数，$\varepsilon$表示服从固定分布的均值为$0$的噪音,$\varepsilon$是random variable。$\odot$表示element-wise乘法。最后的loss函数是关于$\varepsilon$的期望：$\bar{L}(\zeta)=\mathbb{E}\left[L(\theta)\right]$，然后优化相应的$\zeta$，$\varepsilon$不能被优化，因为它是random variable。
一个有$p$个输入单元，$q$个输出单元的fully-connected layer表示如下：
$$y=wx+b \tag{14}$$
其中$w\in \mathbb{R}\^{q\times p}$，$x\in \mathbb{R}\^{p}$,$b\in \mathbb{R}\^{q}$，对应的noisy linear layer定义如下：
$$y=(\mu\^w+\sigma\^w\odot\varepsilon\^w)x + \mu\^b+\sigma\^b\odot\varepsilon\^b \tag{15}$$
就是用$\mu\^w+\sigma\^w\odot\varepsilon\^w$取代$w$，用$\mu\^b+\sigma\^b\odot\varepsilon\^b$取代$b$。其中$\mu\^w,\sigma\^w\in \mathbb{R}\^{q\times p} $，而$\mu\^b,\sigma\^b\in\mathbb{R}\^{q}$是可以学习的参数，而$\varepsilon\^w\in \mathbb{R}\^{p\times q},\varepsilon\^b \in \mathbb{R}\^{q}$是random variable。
作者提出了两种添加noise的方式，一种是Independent Gaussian noise，一种是Factorised Gaussion noise。使用Factorised的原因是减少随机变量的计算时间，这些时间对于单线程的任务来说还是很多的。
#### Independent Gaussian noise
应用到每一个weight和bias的noise都是independent的，对于$\varepsilon\^w$的每一项$\varepsilon\_{i,j}\^w$来说，它们的值都是从一个unit Gaussion distribution中采样得到的；$varepsilon\^b$同理。所以对于一个$p$个输入,$q$个输出的noisy linear layer总共有$pq+q$个noise 变量。

#### Factorised Gaussian noise
通过对$\varepsilon\_{i,j}\^w$来说，可以将其分解成$p$个$\varepsilon_i$用于$p$个输入和$q$个$\varepsilon_j$用于$q$个输出，总共有$p+q$个noiss变量。每一个$\varepsilon\_{i,j}\^w$和$\varepsilon\_{j}\^b$可以写成：
$$\varepsilon\_{i,j}\^w = f(\varepsilon_i)f(\varepsilon_j) \tag{16}$$
$$\varepsilon\_{j}\^b = f(\varepsilon_j)\tag{17}$$
其中$f$是一个实函数，在第一个式子中$f(x) = sng(x)\sqrt{|x|}$，在第二个式子中可以取$f(x)=x$，这里选择了和第一个式子中一致。
因为noisy network的loss函数是$\bar{L}(\zeta)=\mathbb{E}\left[L(\theta)\right]$，是关于noise的一个期望，梯度如下：
$$\nabla\bar{L}(\zeta)=\nabla\mathbb{E}\left[L(\theta)\right]=\mathbb{E}\left[\nabla\_{\mu,\Sigma}L(\mu+\Sigma\odot\varepsilon)\right] \tag{18}$$
使用Monte Carlo估计上述梯度，在每一个step采样一个sample进行optimization:
$$\nabla\bar{L}(\zeta)\approx\nabla\_{\mu,\Sigma}L(\mu+\Sigma\odot\varepsilon) \tag{19}$$

### Noisy DQN and dueling
相对于DQN和dueling DQN来说，noisy DQN and dueling主要做了两方面的改进：
1. 不再使用$\varepsilon$-greedy behaviour policy了，而是使用greedy behaviour policy采样优化randomised action-value function。
2. 网络中的fully connected layers全都换成了参数化的noisy network，noisy network的参数在每一次replay之后从noise服从的distribution中进行采样。这里使用的nose是factorised Gaussian noise。

在replay 整个batch的过程中，noisy network parameter sample保持不变。因为DQN和Dueling每执行一个action step之后都会执行一次optimization，每次采样action之前都要重新采样noisy network parameters。

#### Loss
$Q(s,a,\epsilon;\zeta)$可以看成$\zeta$的一个random variable，NoisyNet-DQN loss如下：
$$\bar{L}(\zeta) = \mathbb{E}\left[\mathbb{E}\_{(x,a,r,y)}\sim D\left[r + \gamma \max\_{b\in A}Q(y, b, \varepsilon';\zeta\^{-}) - Q(x,a,\varepsilon;\zeta)\right]^2\right]\tag{20}$$
其中外层的期望是$\varepsilon$相对于noisy value function $Q(x,a, \varepsilon;\zeta)$和$\varepsilon'$相对于noisy target value function $Q(x,a, \varepsilon';\zeta\^{-}$。对于buffer中的每一个transition，计算loss的无偏估计，只需要计算target value和true value即可，为了让target value和true之间没有关联，target network和online network采用independent noises。
就double dqn中的action选择来说，采样一个新的independent sample $\varepsilon\^{''}$计算action value，然后使用greedy操作，NoisyNet-Dueling的loss如下：
$$\bar{L}(\zeta) = \mathbb{E}\left[\mathbb{E}\_{(x,a,r,y)}\sim D\left[r + \gamma Q(y, b\^{\*}(y), \varepsilon';\zeta\^{-} - Q(x,a,\varepsilon;\zeta)\right]^2\right]\tag{21}$$
$$b\^{\*}(y) = \arg \max\_{b\in A} Q(y, b(y), \varepsilon\^{''};\zeta)\tag{22}$$

### Noisy-A3C
Noisy-A3C相对于A3C有以下的改进：
1. entropy项被去掉了;
2. fully-connected layer被替换成了noisy network。

A3C算法中没有像$\epsilon$-greedy这样进行action exploration，选中的action通常是从current policy中选的，加入entropy是为了鼓励exploration，而不是选择一个deterministic policy。当添加了noisy weights时，对参数进行采样就表示选择不同的current policy，就已经代表了exploration。NoisyNet相当于直接在policy space中进行exploration，而entropy项就可以去掉了。

### Noisy Networks的初始化
在unfactorised noisy networks中，每个$\mu\_{i,j}$从独立的均匀分布$U\left[-\sqrt{\frac{3}{p}}, \sqrt{\frac{3}{p}}\right]$中采样初始化，其中$p$是对应linear layer的输入个数，$\sigma\_{i,j}$设置为一个常数$0.0017$，这是从监督学习的任务中借鉴的。
在factorised noisy netowrks中，每个$\mu\_{i,j}$从独立的均匀分布$U\left[-\sqrt{\frac{1}{p}}, \sqrt{\frac{1}{p}}\right]$中进行采样，$\sigma\_{i,j}$设置为$\frac{\sigma_0}{p}$，超参数$\sigma_0$设置为$0.5$。

### 算法
算法5 NoisyNet-DQN / NoisyNet-Dueling
输入: Env Environment; $\varepsilon$ random variables of the network的集合
输入: DUELING Boolean; "true"代表NoisyNet-Dueling and "false"代表 NoisyNet-DQN
输入: $B$空replay buffer; $\zeta$初始的network parameters; $\zeta\^{-}$初始的target network parameters
输入: replay buffer大小$N_B$; batch size $N_T$; target network更新频率$N\^{-}$
输出: $Q(\cdot, \varepsilon; \zeta)$ action-value function
**for** episode $e\in  {1,\cdots , M}$ do
$\qquad$初始化state sequence $x_0 \sim Env$
$\qquad$**for** $t \in {1,\cdots }$ do
$\qquad\qquad$设置$x \leftarrow x_0$
$\qquad\qquad$采样 a noisy network  $\xi\sim \varepsilon$
$\qquad\qquad$选择an action $a \leftarrow \arg \max\_{b\in A} Q(x, b, \xi; \zeta)$
$\qquad\qquad$采样 next state $y \sim  P (\cdot|x, a)$, 接收 reward $r \leftarrow R(x, a) $以及$x_0 \leftarrow y$
$\qquad\qquad$将transition (x, a, r, y)添加到replay buffer
$\qquad\qquad$**if** $|B| \gt N_B$ then
$\qquad\qquad\qquad$删掉最老的transition
$\qquad\qquad$**end if**
$\qquad$采样一个大小为$N_T$的batch, transitions $((x_j, a_j, r_j, y_j) \sim D)\_{j=1}\^{N_T}$
$\qquad\qquad$采样noisy variables用于online network $\xi \sim\varepsilon$
$\qquad\qquad$采样noisy variables用于target network $\xi'\sim\varepsilon$
$\qquad\qquad\qquad$**if** DUELING then
$\qquad\qquad\qquad$采样noisy variables用于选择action的network $\xi\sim\varepsilon$ 
$\qquad\qquad$**end if**
$\qquad\qquad$**for** $j \in {1,\cdots, N_T}$ do
$\qquad\qquad\qquad$**if** $y_j$ is a terminal state then
$\qquad\qquad\qquad\qquad$$\hat{Q}\leftarrow r_j$
$\qquad\qquad\qquad$**end if**
$\qquad\qquad\qquad$**if** DUELING then
$\qquad\qquad\qquad\qquad b\^{\*}(y_j) = \arg \max\_{b\in A} Q(y_j, b, \xi\^{''}; \zeta)$
$\qquad\qquad\qquad\qquad\qquad \hat{Q}\leftarrow r_j + \gamma Q(y_j, b\^{\*}(y_j), \xi';\zeta\^{-})$
$\qquad\qquad\qquad$**else**
$\qquad\qquad\qquad\qquad$$\hat{Q}\leftarrow r_j + \gamma \max\_{b\in A} Q(y_j, b, \xi';\zeta\^{-})$
$\qquad\qquad$**end if**
$\qquad\qquad\qquad$利用loss $(\hat{Q}-Q(x_j,a_j, \xi;\zeta))^2$的梯度更新$\zeta$
$\qquad\qquad$**end for**
$\qquad\qquad$每隔$N\^{-}$步更新target network:$ \zeta\^{−}\leftarrow \zeta$ 
$\qquad$**end for**
**end for**

算法6 NoisyNet-A3C for each actor-learner thread
输入: Environment Env, 全局共享参数$(\zeta\_{\pi},\zeta\_{V})$ , 全局共享counter $T$和maximal time $T\_{max}$ 
输入: 每个线程的参数 $(\zeta'\_{\pi},\zeta'\_{V})$, random variables $\varepsilon$的集合, 每个线程的counter $t$和TD-$\gamma$的长度$t\_{max}$
输出: policy $\pi(\cdot; \zeta\_{\pi}, \varepsilon)$和value $V(\cdot; \zeta\_{V}, \varepsilon)$
初始化线程counter $t \leftarrow 1$
**repeat**
$\qquad$重置acumulative gradients: $d\zeta\_{\pi}\leftarrow 0$和$d\zeta_V \leftarrow 0$
$\qquad$Synchronise每个线程的parameters: $\zeta'\_{\pi}\leftarrow \zeta\_{\pi}$和$\zeta\_V\leftarrow \zeta\_V$
$\qquad$counter $\leftarrow 0$
$\qquad$从Env中得到state $x_t$
$\qquad$采样noise: $\xi\sim\varepsilon$
$\qquad r \leftarrow \[\]$
$\qquad a \leftarrow \[\]$
$\qquad x \leftarrow \[\]$和$x\[0\] \leftarrow x_t$
$\qquad$**repeat**
$\qquad\qquad$采样action: $a_t \sim\pi(\cdot|x_t;\zeta'\_{\pi};\xi)$
$\qquad\qquad$$a[−1]\leftarrow a_t$
$\qquad\qquad$接收reward $r_t$和next state $x\_{t+1}$
$\qquad\qquad$$r[−1]\leftarrow r_t$和$x[−1]\leftarrow x_t+1$
$\qquad\qquad$$t\leftarrow t + 1$和 $T\leftarrow T + 1$
$\qquad\qquad$$counter = counter + 1$
$\qquad\qquad$**until** $x_t\ \ terminal\ \ or\ \ counter == t\_{max} + 1$
$\qquad$**if** $x_t$ is a terminal state then
$\qquad\qquad$$Q = 0$
$\qquad$**else**
$\qquad\qquad$$Q = V(x_t; \zeta'\_{V}, \xi)$
$\qquad$**end if**
$\qquad$**for** $i \in \{counter − 1, \cdots, 0\}$ do
$\qquad\qquad$更新Q: $Q\leftarrow r[i] + \gamma Q$
$\qquad\qquad$累积policy-gradient: $d\zeta\_{\pi} \leftarrow d\zeta\_{\pi} + \nabla \zeta'\_{\pi}log(\pi(a[i]|x[i]; \zeta'\_{\pi}, \xi))[Q − V(x[i]; \zeta'\_{\pi}V, \xi)]$
$\qquad\qquad$累积 value-gradient: $d\zeta_V \leftarrow ← d\zeta_V+ \nabla \zeta'\_{V}[Q − V(x[i]; \zeta'\_{V}, \xi)]^2$
$\qquad$**end for**
$\qquad$执行$\zeta\_{\pi}$的asynchronous update: $\zeta\_{\pi}\leftarrow \zeta\_{\pi} + \alpha\_{\pi}d\zeta\_{\pi}$
$\qquad$执行$\zeta\_{V}$的asynchronous update: $\zeta\_{V}\leftarrow \zeta\_{V} − \alpha_VdV\zeta\_{V}$
**until** $T \gt T\_{max}$

## Rainbow

## 参考文献
1.https://blog.csdn.net/yangshaokangrushi/article/details/79774031
2.https://link.springer.com/article/10.1007%2FBF00992698
3.https://www.jianshu.com/p/b92dac7a4225
4.https://datascience.stackexchange.com/questions/20535/what-is-experience-replay-and-what-are-its-benefits/20542#20542
5.https://stats.stackexchange.com/questions/897/online-vs-offline-learning
6.https://www.freecodecamp.org/news/improvements-in-deep-q-learning-dueling-double-dqn-prioritized-experience-replay-and-fixed-58b130cc5682/
7.https://jaromiru.com/2016/11/07/lets-make-a-dqn-double-learning-and-prioritized-experience-replay/
8.https://datascience.stackexchange.com/questions/32873/prioritized-replay-what-does-importance-sampling-really-do
9.https://papers.nips.cc/paper/5249-weighted-importance-sampling-for-off-policy-learning-with-linear-function-approximation.pdf