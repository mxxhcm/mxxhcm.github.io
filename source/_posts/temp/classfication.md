---
title: classfication
date: 2018-10-21 18:47:44
tags:
 - classfication
 - 分类
 - 机器学习
categories: 机器学习
mathjax: true
---

## Classfication

## LDA

## Logistic Regression

### Logistic function
$$ S(x) = \frac{1}{1+e^{x} }$$
如下图所示：
![logistic_func](logistic_function.png)
它的取值在$[0,1]$之间。
logistic regression的目标函数是：
$$h(x) = \frac{1}{1+e^{-\theta^T x}}$$
其中$x$是输入，$\theta$是要求的参数。

### 思路
Logistic regression利用logistic function进行分类，给出一个输入，经过参数$\theta$的变换，输出一个$[0,1]$之间的值，如果大于$0.5$，把它分为一类，小于$0.5$，分为另一类。这个$0.5$只是一个例子，可以根据不同的需求选择不同的值。
$\theta^T x$相当于给出了一个非线性的决策边界。

### Cost function
给出两种方式推导logistic regression的cost function

#### Maximum likelyhood estimation

#### Cross-entropy
对于$k$类问题，写出交叉熵公式如下所示：
$$J(\theta) = -\frac{1}{n}\left[\sum\_{i=1}^m \sum_k y_k^{(i)} \log h(x_k^{(i)} ) \right]$$ 
当$k=2$时：
$$J(\theta) = -\frac{1}{n}\left[\sum\_{i=1}^m  y^{(i)} \log h(x^{(i)} ) + (1-y^{(i)}) \log (1-h(x^{(i)} ))\right]$$ 

### 梯度下降
$$\nabla J =


## 参考文献
1.https://zhuanlan.zhihu.com/p/28408516
