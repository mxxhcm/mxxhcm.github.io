---
title: generative model and discriminative model
date: 2019-10-26 19:24:35
tags:
 - 机器学习
 - generative model
 - 生成式模型
 - discriminative model
 - 判别式模型
categories: 机器学习
mathjax: true
---

## 生成式模型
$$p(Y|X) = \frac{p(Y) p(X|Y)}{p(X)} = \frac{p(X|Y)p(Y)}{\sum\_{Y'} p(Y')p(X|Y')}$$
生成式模型为了对$X$进行分类，会对如何生成数据进行建模，学习联合概率密度函数$P(X,Y)$。然后基于学习的联合密度函数$P(X,Y)$，预测哪一类最有可能。而这也是为什么他叫生成式模型，因为它是根据联合概率$P(X,Y)$和已知的先验分布来计算后验分布$P(Y|X)$进行分类，而$P(X,Y)$可以看成生成样本$(X,Y)$的概率。
常见的生成模型有：朴素贝叶斯和隐马尔可夫。

## 判别式模型
$$P(Y|X)$$
判别式模型直接求的就是$P(Y|X)$即给定输入$X$的条件概率。不关心数据如何产生，只关心怎么分类就行了。而判别式模型，是直接根据$X$给出$Y$的判别。
常见的判别式模型有KNN，感知机，逻辑回归，决策树，最大熵，boost以及CRF等。

## 生成式模型vs判别式模型
- 生成式模型和判别式模型都是用于分类问题的。分类问题的目标是给出输入$X$，预测它的类别$Y$，即：
$$Y=f(X)$$
或者
$$P(Y|X)$$
- 生成模型是利用数据学习联合概率密度分布，而判别模型是利用数据学习条件概率分布。
- 存在隐变量时候，可以使用生成式不能使用判别式
- 生成式方法可以计算联合概率分布$P(X, Y)$


## 参考文献
1.https://www.quora.com/What-are-the-differences-between-generative-and-discriminative-machine-learning
2.https://www.zhihu.com/question/20446337/answer/256466823
