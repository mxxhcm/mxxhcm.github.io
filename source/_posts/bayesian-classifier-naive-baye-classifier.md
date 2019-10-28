---
title: beyesian classifier naive baye classifier
date: 2019-10-27 21:57:09
tags:
 - 朴素贝叶斯分类器
 - 概率论与统计
 - 贝叶斯分类器
categories: 机器学习
---

## 朴素贝叶斯分类器
朴素贝叶斯法将实例分到后验概率最大的类中，相当于期望风险最小化。

朴素贝叶斯有一个假设：条件独立假设，拿公式举个例子就是
$$P(X|Y=y_k)=\prod\_{i=1}^m P(x_i|Y=y_k) \tag{1}$$
其中$X=[x_1, x_2,\cdots, x_m]$即$m$个属性。
贝叶斯分类器的流程如下所示：
1. 给出训练样本集，属性集合$X=[x_1, x_2,\cdots, x_m]$，标签集合$Y=[y_1, y_2,\cdots, y_n]$，计算每个类别$y_j$中出现属性$x_i$的条件概率，即
$$P(x_i|y_j), 1 \le i \le m, 1 \le j \le n \tag{2}$$
2. 给出一个新的样本$X$，根据贝叶斯定理以及条件独立假设，计算：
$$P(y_k|X) = \frac{P(y_k)P(X|y_k)}{P(X)} = \frac{P(y_k) \prod\_{i=1}^m P(x_i|y_k) }{P(X)} \tag{3}$$
3. 从$P(y_k|X)$中选出最大的$P$对应的$y_k$当做label。

可以看出，朴素贝叶斯分类器的关键就是计算条件概率：
- 当属性$X$是离散值时，可以统计样本中各个$P(x_i|y_j)$的频率近似计算概率
- 当属性$X$是连续值时，可以假设变量服从某种分布，使用训练数据估计分布的参数。 如高斯分布的均值和方差。

## 参数估计方法
可以使用[贝叶斯估计](https://mxxhcm.github.io/2019/07/31/probability_basic/)，[最大似然估计](https://mxxhcm.github.io/2019/01/20/maximum-likelyhood-estimation/)和[最大后验估计](https://mxxhcm.github.io/2019/07/31/probability_basic/)求朴素贝叶斯的参数。

## 参考文献
1.https://www.cnblogs.com/phoenixzq/p/3539619.html
2.http://funhacks.net/2015/05/18/Bayesian-classifier/
