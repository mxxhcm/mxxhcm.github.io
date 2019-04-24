---
title: Asynchronous Methods for Deep Reinforcement Learning
date: 2019-04-19 18:11:56
tags:
 - 强化学习
 - A3C
categories: 强化学习
---

## 摘要
DQN使用experience replay buffer来稳定学习过程。本文提出了一个新的框架，使用异步的梯度下降稳定学习过程。这个框架既适用于on-policy，也适用于off-policy环境上，即能应用于离散动作空间，也能应用于连续的动作空间，既能训练前馈智能体，也能训练循环智能体。

## Introduction
Online学习是不稳定的，并且online更新是强相关的。DQN通过引入experience replay buffer解决了这个问题，但是DQN只能应用在off policy算法上。总的来说，DQN通过引入replay buffer取得了很大成功，但是replay buffer还有以下的几个缺点：
- 在每一步交互的时候使用了更多的内存和计算资源
- 它只能应用在off policy的算法上，也就是说权重的更新可能会使用到很久之前的数据。

这篇文章提出不使用replay buffer，而是使用异步的框架，同时在多个相同的环境中操作多个智能体（每个环境中一个智能体）并行的采集数据。这种并行性也能将智能体的数据分解成更稳定的过程（即和experience replay buffer起到了相同的作用），因为在给定的一个时间步，智能体可能会experience很多个不同的states。
这个框架既可以应用在on policy算法，如Sarsa，n-step methods和actor-critc等方法上，也可以应用在off policy算法如Q-learning上。

## 异步框架
作者给出了一个框架，将够将on-policy search的actor-critic方法以及off-policy value-based的Q-learning方法都包括进去。
具体的，使用一台机器上的多CPU线程，这样子可以避免在不同机器上传递参数和梯度的消耗。然后，多个并行的actor-learner可能会探索环境的不同部分，每个actor-learner可以设置不同的exploration policy。不同的thread运行不同的exploration policy，多个actor-learner并行执行online update可能比单智能体更新在时间上更不相关。所以这里使用了不同的探索策略取代了DQN中buffer稳定学习过程的作用。
除了稳定学习过程之外，多个actor-learner还可以减少训练时间，此外，不使用buffer以后还可以使用on-policy的方法进行训练。

**总的来说，下面要介绍的四个算法，前面三个算法都使用了target network，第四个A3C算法没有使用target network。最重要的是所有四个算法都使用了多个actor-learner进行训练，并且使用累计的梯度进行更新（相当于batch的作用）。总共出现了三类参数，一类是network参数，一类是target network参数，一类是thread-specific（每个线程的参数）的参数。thread-specific参数是每个线程自己持有的，通过更新每个线程的参数更新network的参数，然后使用network的参数更新target network的参数，target network参数比network参数更新的要慢很多。
A3C算法的实质就是在多个线程中同步训练。分为主网络和线程中的网络，主网络不需要训练，主要用来存储和传递参数，每个线程中的网络用来训练参数。总的来说，多个线程同时训练提高了效率，另一方面，减小了数据之间的相关性，比如，线程$1$和$2$中都用主网络复制来的参数计算梯度，但是同一时刻只能有一个线程更新主网络的参数，比如线程$1$更新主网络的参数，那么线程$2$利用原来主网络参数计算的梯度会更新在线程$1$更新完之后的主网络参数上。**

### 异步的one-step Q-learning
- 每个thread都和它自己的环境副本进行交互，在每一个时间步计算Q-learning loss的梯度。
- 通过使用不同的exploration策略，可以改进性能，这里实现exploration policy不同的方式就是使用$\epsilon$的不同取值实现。
- 使用一个共享的更新的比较缓慢的target network，就是和DQN中的target network一样。
- 同时也使用多个时间步上的累计梯度，和batch挺像的，这就减少了multi actor learner重写其他更新的可能性，同时也在计算效率和数据效率方面做了一个权衡。

#### 伪代码
**Algorithm 1** 异步的one-step Q-learning－－每个actor-learn线程的伪代码
用$\theta,\theta^{-}$表示全局共享参数，计数器$T=0$，
初始化线程时间步计数器$t\leftarrow 0$，
初始化target network权重$\theta^{-} \leftarrow 0$,
初始化network梯度$d\theta\leftarrow 0$，
初始化，得到初始状态$s$，
**repeat**
$\qquad$使用$\epsilon-$greedy策略采取action $a$，
$\qquad$接收下一个状态$s'$和reward $r$，
$\qquad$设置target value，$y=\begin{cases}r,&for\ terminal\ s' \\r+\gamma max_{a'}Q(s',a';\theta^{-}), &for\ non-terminal\ s'\end{cases}$
$\qquad$累计和$\theta$相关的梯度：$d\theta \leftarrow d\theta+\frac{\partial (y-Q(s,a;\theta))^2}{\partial \theta}$
$\qquad s\leftarrow s'$
$\qquad T\leftarrow T+1, t\leftarrow t+1$
$\qquad$**if** $T\ \ mod\ \ I_{target} ==0 $，那么
$\qquad\qquad$更新target network $\theta^{-}\leftarrow 0$
$\qquad$**end if**
$\qquad$**if** $t\ \ mod\ \ I_{AsyncUpdate} ==0$或者$s$是terminal state，那么
$\qquad\qquad$使用$d\theta$异步更新$\theta$
$\qquad\qquad$将累计梯度$d\theta\leftarrow 0$
$\qquad$**end if**
**until** $T\ge T_{max}$

### 异步的one-step Sarsa
#### 概述
- 和算法$1$很像，$Q-learning$计算target value使用$r+\gamma max_{a'}Q(s',a';\theta^{-})$，而Sarsa计算target value使用$r+\gamma Q(s',a';\theta^{-})$，即Q-learning的bahaviour policy和评估的策略是不一样的，而Sarsa的behaviour policy和评估策略是一样的。
- 使用target network，
- 同时使用多个时间步的累计梯度更新用来稳定学习过程。

#### 伪代码
和算法$1$很像。

### 异步的n-step Q-learning
#### 概述
- 计算$n-step$的return
- 在计算一次更新的时候，使用exploration policy采样到$t_{max}$步或者到terminal state。然后累加从上次更新到$t_{max}$时间步的reward。
- 然后计算$n-step$更新对于上次更新之后所有state-action的梯度。
- 使用单个时间步中的累计梯度进行更新。
- 使用了target network。

#### 伪代码
**Algorithm 2** 异步的n-step Q-learning算法－－每个actor-learner线程的伪代码
用$\theta,\theta^{-}$表示全局共享的network参数和target network参数，用$T=0$表示全局共享计数器。
初始化线程步计数器$t\leftarrow 1$，
初始化target network参数$\theta^{-}\leftarrow \theta$
初始化每个线程的参数参数$\theta^{-}\leftarrow \theta$
初始化网络梯度$d\theta\leftarrow 0$
**repeat**
$\qquad$重置累计梯度$d\theta\leftarrow0$
$\qquad$同步每个线程的参数$\theta'=\theta$
$\qquad t_{start}=t$
$\qquad$得到$s_t$
$\qquad$**repeat**
$\qquad\qquad$根据基于$Q(s_t,a;\theta')$的$\epsilon-greedy$策略执行动作$a_t$，
$\qquad\qquad$接收下一个状态$s_{t+1}$和reward $r_t$，
$\qquad\qquad T\leftarrow T+1, t\leftarrow t+1$
$\qquad$ **until** terminal $s_t$或者$t-t_{start}==t_{max}$
$\qquad$设置奖励$R=\begin{cases}0,&for\ terminal\ s_t\\max_aQ(s_t,a;\theta^{-}), &for\ non-terminal\ s_t\end{cases}$
$\qquad$**for** $i\in\{t-1,\cdots,t_{start}\}$ do
$\qquad\qquad R\leftarrow r_i+\gamma R$
$\qquad\qquad$累计和$\theta'$相关的梯度：$d\theta \leftarrow d\theta+\frac{\partial (R-Q(s_t,a;\theta'))^2}{\partial \theta'}$
$\qquad$**end for**
$\qquad$使用$d\theta$异步更新$\theta$.
$\qquad$**if**$\quad T\quad mod\quad I_{target}==0$那么
$\qquad\qquad\theta^{-}\leftarrow \theta$
$\qquad$**end if**
**until** $T\gt T_{max}$

### 异步的advantage actor-critic
#### 概述
- A3C算法，是一个actor-critic方法，使用值函数$V(s_t;\theta_v)$辅助学习policy $\pi(a_t|s_t;\theta)$，同时这里使用$n-step$的returns更新policy和value function。
- 每隔$t_{max}$个action更新一次或者到了terminal state更新一次。
- Actor的更新方向为$\nabla_{\theta'}log\pi(a_t|s_t;\theta')A(s_t,a_t;\theta,\theta_v)$，其中$A$是advantage function的一个估计，通过$\sum_{i=0}^{k-1}\gamma^ir_{t+i}+\gamma^kV(s_{t+k};\theta_v) - V(s_t;\theta_v)$计算。
- 这里同样使用并行的actor-learner和累计的梯度用来稳定学习。$\theta$和$\theta_v$在实现上通常共享参数。
- 添加entropy正则项鼓励exploration。包含了正则化项的的objective function的梯度为$\nabla_{\theta'}log\pi(a_t|s_t;\theta')(R_t-V(s_t;\theta_v))+\beta\nabla_{\theta'}H(\pi(s_t;\theta'))$。这里的$R$就是上面的$\sum_{i=0}^{k-1}\gamma^ir_{t+i}+\gamma^kV(s_{t+k};\theta_v) - V(s_t;\theta_v)$。
- Critic的更新方向通过最小化loss来实现，这里的loss指的是TD-error，即$\sum_{i=0}^{k-1}\gamma^ir_{t+i}- V(s_t;\theta_v)$。
- 没有使用target network。

#### 伪代码
**Algorithm 3** 异步的actor-critic－－每个actor-learn线程的伪代码
用$\theta,\theta_v$表示全局共享参数，用$T=0$表示全局共享计数器，
用$\theta',\theta'_v$表示每个线程中的参数
初始化线程步计数器$t\leftarrow 1$，
**repeat**
$\qquad$重置梯度$d\theta\leftarrow 0,d\theta_v\leftarrow 0$，
$\qquad$同步线程参数$\theta'=\theta,\theta'_v=\theta_v$
$\qquad t_{start}=t$
$\qquad$得到状态$s_t$，
$\qquad$**repeat** 
$\qquad\qquad$根据策略$\pi(a_t|s_t;\theta')$执行动作$a_t$，
$\qquad\qquad$接收下一个状态$s_{t+1}$和reward $r_t$，
$\qquad\qquad T\leftarrow T+1, t\leftarrow t+1$
$\qquad$ **until** terminal $s_t$或者$t-t_{start}==t_{max}$
$\qquad$设置奖励$R=\begin{cases}0,&for\ terminal\ s_t\\V(s_t,\theta'_v), &for\ non-terminal\ s_t\end{cases}$
$\qquad$**for** $i\in\{t-1,\cdots,t_{start}\}$ do
$\qquad\qquad R\leftarrow r_i+\gamma R$
$\qquad\qquad$累计和$\theta'$相关的梯度：$d\theta \leftarrow d\theta+\frac{\partial (y-Q(s,a;\theta))^2}{\partial \theta}$
$\qquad\qquad$累计和$\theta'_v$相关的梯度：$d\theta_v \leftarrow d\theta_v+\frac{\partial (R-V(s_i;\theta'_v))^2}{\partial \theta'_v}$
$\qquad$**end for**
$\qquad$使用$d\theta$异步更新$\theta$，使用$d\theta_v$异步更新$\theta_v$.
**until** $T\ge T_{max}$

### 优化方法
作者尝试了三种不同的优化方法，带有momentum的SGD，带有共享statistics的RMSProp以及不带shared statistics的RMSProp。


## 实验
### 优化细节
作者在异步框架中测试了两个优化算法SGD和RMSProp，并且因为效率原因没有使用线程锁。
### 设置
- Atari环境中，每个实验使用$16$个actor-learner线程。
- 所有方法都每隔$5$个actions更新一次，并且使用共享的RMSProp进行优化。
- 三个异步的value-based算法使用每隔$40000$帧更新的共享target network，
- 使用了DQN中action repeat of $4$.
- 网络架构和DQN一样
- 基于值的方法只有一个线性输出层，每个输出单元代表一个action的值。
- actor-critic方法有两个输出层，一个softmax表示选择某一个action的概率，一个线性输出代表值函数。
- 所有实验使用的$\gamma=0.99$，RMSProp的衰减因子$\alpha = 0.99$。
- Value-based方法采用的exploration rate $\epsilon$有是三个取值$\epsilon_1,\epsilon_2,\epsilon_3$，相应的概率为$0.4,0.3,0.3$，它们的值在前$4$百万帧中从$1$退火到$0.1,0.01,0.5$。
- A3C使用了entropy进行正则化，entropy项的权重为$\beta=0.01$
- 初始学习率从分布$LogUniform(10^{-4},10^{-2})$中进行采样，在训练过程中退火到$0$。

## Discussion
