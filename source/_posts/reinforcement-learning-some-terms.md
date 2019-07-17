---
title: reinforcment learning的一些术语
date: 2019-05-29 17:59:28
tags:
- online
- offline
- on-policy
- off-policy
- bootstrap
- 强化学习
categories: 强化学习
---

## 分类方式
### online vs offline
online方法中训练数据一直在不断增加，基本上强化学习都是online的，而监督学习是offline的。

### on-policy vs off-policy
behaviour policy是采样的policy。
target policy是要evaluation的policy。
behaviour policy和target policy是不是相同的，相同的就是on-policy，不同的就是off-policy，带有replay buffer的都是off-policy的方法。

## bootstrap
当前value的计算是否基于其他value的估计值。
常见的bootstrap算法有DP，TD-gamma
MC算法不是bootstrap算法。


## value-based vs policy gradient vs actor-critic 

### value-based
values-based方法主要有policy iteration和value iteration。policy iteration又分为policy evaluation和policy improvement。
给出一个任务，如果可以使用value-based。随机初始化一个policy，然后可以计算这个policy的value function，这就叫做policy evaluation，然后根据这个value function，可以对policy进行改进，这叫做policy improvement，可以证明policy一定会更好。policy evaluation和policy improvement交替迭代，在线性case下，收敛性是可以证明的，在non-linear情况下，就不一定了。
policy iteraion中，policy evaluation每一次都要进行收敛后才进行policy improvemetn，如果policy evalution只进行一次，然后就进行一次policy improvemetn的话，也就是policy evalution的粒度变小后，就是value iteration。

### policy gradient
value-based方法只适用于discrete action space，对于contionous action space的话，就无能为力了。这个时候就有了policy gradient，给出一个state，policy gradient给出一个policy直接计算出相应的action，然后给出一个衡量action好坏的指标，直接对policy的参数求导，最后收敛之后就求解出一个使用与contionous的policy

### actor-critic
如果policy gradient的metrics选择使用value function，一般是aciton value function的话，我们把这个value function叫做critic，然后把policy叫做actor。通过value funciton Q对policy的参数求导进行优化。
critic跟policy没有关系，而critic指导actor的训练，通过链式法则实现。critic对a求偏导，a对actor的参数求偏导。

## 参考文献
1.https://stats.stackexchange.com/questions/897/online-vs-offline-learning
