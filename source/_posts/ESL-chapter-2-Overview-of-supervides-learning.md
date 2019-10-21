---
title: ESL chapter 2 Overview of supervised learning
date: 2019-01-05 09:30:55
tags:
 - 统计
 - 机器学习
 - 监督学习
 - 非监督学习
categories: 机器学习
mathjax: true
---

## 引言
在机器学习领域，监督学习(supervised learning)的每一个样本都由输入(inputs)和输出(outputs)组成。监督学习的目标就是根据inputs的值去预测outpus的值。
在统计学(statistical)中，inputs通常被称为预测器？？(predictors)，或者叫自变量(independent variables)。
在模式识别(pattern recognition)领域，inputs通常称为特征(features)，或者叫因变量(dependent variables)

## 变量类型和一些术语(terminology)
不同的问题中，输出也不一样。血糖预测问题中，输出是一个定量的(quantitative)测量。手写数字识别问题中，输出是十个不同的类，是定性的(qualitative)，定性的输出也通常被称为类别(catrgorical)，这里的类别是无序的。通常，预测定量的输出被称为回归问题(regression)，预测定性的输出被称为分类问题。这两个问题很相像，多可以看成函数拟合。第三种输出是有序类别，像小，中，大，没有合适的度量表示，因为中和小之间的差别和中和大之间的差别是不同的。
定性分析在代码实现中进行二值化数值表示。即如果只有两类的话，用一个二进制位$0$或者$1$表示，或者$1$和$-1$。当超过两类的时候，通常用虚拟变量(dummy variables)来表示，一个$K$级变量是一个长度为$K$的二进制位，每一个时刻只有一位被置一。
一些常用的表示，$X$表示inputs，$Y$表示定量outputs，$G$表示定性outputs。大写字母表示通用的表示，观测值用小写字母表示，inputs $X$的第$i$个观测值用$x_i$表示，其中$x_i$是一个标量或者向量。矩阵用粗体的大写字母表示，如具有$N$个$p$维向量$x_i, j= 1,\cdots,N$的$N\times p$矩阵$\mathbf{X}$。所有的向量都用的是列向量表示，$\mathbf{A}$的第$i$行是$x_i^T$，第$i$列的转置。

