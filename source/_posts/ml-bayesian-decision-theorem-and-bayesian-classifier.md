---
title: bayesian decision theorem and bayesian classifier 
date: 2019-10-26 20:02:39
tags:
 - 机器学习
categories: 机器学习
mathjax: true
---

## 关键字
贝叶斯分类器，贝叶斯定理，朴素贝叶斯，贝叶斯错误率

## 贝叶斯决策理论
在[概率论基础中](https://mxxhcm.github.io/2019/07/31/probability_basic/)已经简单介绍了频率学派和贝叶斯学派。贝叶斯学派中几个重要的概念：先验，后验。
贝叶斯决策理论是通过计算后验概率进行决策的，贝叶斯决策理论的实际意义不大，但是具有非常重要的理论意义，尤其是误差上界的计算。

### 贝叶斯定理（公式）
假设X,Y是一对随机变量，它们的联合概率$P(X=x, Y=y)$是指$X$取值$x$且$Y$取值为$y$的概率，条件概率是指一个随机变量在另一个随机变量取值已知的情况下取某一个特定值的概率。比如$P(Y=y|X=y)$是指在变量$X$取值$x$的情况下，变量$Y$取值$y$的概率。$X$和$Y$的联合概率和条件概率满足如下关系：
$$P(X,Y) = P(Y|X)\cdot P(X) = P(X|Y)\cdot P(Y) \tag{1}$$
由上面的公式可以得到下面公式，称为贝叶斯定理：
$$ P(Y|X) = \frac{P(X|Y)P(Y)}{P(X)} \tag{2}$$

贝叶斯公式最伟大之处就在于将先验概率转化为了后验概率。使得后验概率变得可以求解了。

### 最小错误率贝叶斯决策

### 最小风险贝叶斯决策

## 贝叶斯分类器
根据贝叶斯决策理论得到的分类器，以贝叶斯定理为基础，是一类分类算法的总称，叫做贝叶斯分类器。常见的贝叶斯分类器有[朴素贝叶斯](http://mxxhcm.github.io/2019/10/27/bayesian-classifier-naive-baye-classifier/)和[贝叶斯信念网络](http://mxxhcm.github.io/2019/01/06/bayesian-classifier-bayesian-networks/)。

### 贝叶斯决策边界
贝叶斯分类器产生最低的错误率，叫做贝叶斯错误率。因为贝叶斯分类器总是选择使式子$(1)$最大化的类，在$X=x_0$处的错误率是$1-\max_j Pr(Y=j|X=x_0)$。
整个贝叶斯分类器的错误率是：
$$1-\mathbb{E}\left(\max_j Pr(Y=j|X=x_0)\right) \tag{6}$$
其中期望表示计算所有$X$的可能取值上的平均错误率。

## 参考文献
1.https://www.cnblogs.com/phoenixzq/p/3539619.html
2.http://funhacks.net/2015/05/18/Bayesian-classifier/