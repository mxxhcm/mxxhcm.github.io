---
title: tensorflow softmax
date: 2019-05-16 09:04:48
tags:
 - tensorflow 
 - python
categories: tensorflow
mathjex: true
---

## 各种softmax
- tf.nn.softmax
- tf.nn.log_softmax
- tf.nn.softmax_cross_entropy_with_logits
- tf,sparse_softmax_cross_entropy_with_logits

## logits
什么是logits
### 数学上
假设一个事件发生的概率为 p，那么该事件的logits为$\text{logit}(p) = \log\frac{p}{1-p}$.
### Machine Learning中
logits在机器学习中就是前向传播网络的分类结果，是未归一化的概率，总和不为$0$，输入softmax函数之后可以得到归一化的概率。

## tf.nn.softmax
### API
``` python
tf.nn.softmax(
	logits,
	axis=None,
	name=None
)
```
### 功能
上面函数实现了如下的功能：
softmax = tf.exp(logits) / tf.reduce_sum(tf.exp(logits), axis)
就是将输入的logits经过softmax做归一化。

## tf.nn.log_softmax
### API
``` python
tf.nn.softmax(
	logits,
	axis=None,
	name=None
)
```

### 功能
该函数实现了如下功能。
logsoftmax = logits - log(reduce_sum(exp(logits), axis))

## tf.nn.softmax_cross_entropy_with_logits
### API
``` python
tf.nn.softmax_cross_entropy_with_logits(
    _sentinel=None,  # pylint: disable=invalid-name
    labels=None,
    logits=None,
    dim=-1,
    name=None
)
```
### 功能
计算logits经过softmax之后和labels之间的交叉熵

## tf.sparse_softmax_cross_entropy_with_logits
### API
``` python
tf.nn.sparse_softmax_cross_entropy_with_logits(
    _sentinel=None,  # pylint: disable=invalid-name
    labels=None,
    logits=None,
    name=None
)
```

### 功能
计算logits和labels之间的稀疏softmax交叉熵

## 参考文献
1.http://landcareweb.com/questions/789/shi-yao-shi-logits-softmaxhe-softmax-cross-entropy-with-logits
2.https://stackoverflow.com/questions/41455101/what-is-the-meaning-of-the-word-logits-in-tensorflow
3.https://stackoverflow.com/a/43577384
4.https://stackoverflow.com/a/47852892
5.https://www.tensorflow.org/tutorials/estimators/cnn
6.https://www.zhihu.com/question/60751553
