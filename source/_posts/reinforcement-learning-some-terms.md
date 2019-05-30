---
title: reinforcment learning的不同分类方法
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

## 参考文献
1.https://stats.stackexchange.com/questions/897/online-vs-offline-learning
