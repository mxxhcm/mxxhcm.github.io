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
- tf.nn.softmax。
- tf.nn.log_softmax。
- tf.nn.softmax_cross_entropy_with_logits_v2中label是用稀疏的（one-hot）表示的。
- tf.nn.sparse_softmax_cross_entropy_with_logits中label是非稀疏的。

## 对比
tf.nn.softmax()
tf.nn.log_softmax()
tf.nn.softmax_cross_entropy_with_logits_v2()
tf.nn.sparse_cross_entropy_with_logits()

## logits
什么是logits
### 数学上
假设一个事件发生的概率为 p，那么该事件的logits为$\text{logit}(p) = \log\frac{p}{1-p}$.
### Machine Learning中
深度学习中的logits和数学上的logits没有太大联系。logits在机器学习中前向传播的输出，是未归一化的概率，总和不为$1$。将logits的输出输入softmax函数之后可以得到归一化的概率。

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

### 示例
``` python
import tensorflow as tf

logits = [1.0, 1.0, 2.0, 2.0, 2.0, 2.0]
res_op = tf.nn.softmax(logits)
sess = tf.Session()
result = sess.run(res_op)
print(result)
print(sum(result))
# output
# [0.07768121 0.07768121 0.21115941 0.21115941 0.21115941 0.21115941]
# 1.0000000447034836
# 因为有指数运算，所以就不是整数
```

## tf.nn.log_softmax
### API
``` python
tf.nn.log_softmax(
	logits,
	axis=None,
	name=None
)
```

### 功能
该函数实现了如下功能。
logsoftmax = logits - log(reduce_sum(exp(logits), axis))

### 示例
``` python
import tensorflow as tf

logits = [1.0, 1.0, 2.0, 2.0, 2.0, 2.0]
res_op = tf.nn.log_softmax(logits)
sess = tf.Session()
result = sess.run(res_op)
print(result)
print(sum(result))

# output
# [-2.555142  -2.555142  -1.5551419 -1.5551419 -1.5551419 -1.5551419]
# -11.330851554870605
```

## tf.nn.softmax_cross_entropy_with_logits_v2
### API
``` python
tf.nn.softmax_cross_entropy_with_logits_v2(
    labels, # shape是[batch_size, num_calsses]，每一个labels[i]都应该是一个有效的probability distribution
    logits, # 没有normalized的log probabilities
    axis=None,
    name=None
    dim=-1,
)
```
### 功能
计算logits经过softmax之后和labels之间的交叉熵

## tf.sparse_softmax_cross_entropy_with_logits
### API
``` python
tf.nn.sparse_softmax_cross_entropy_with_logits(
    _sentinel=None,  # pylint: disable=invalid-name
    labels=None,    # shape是[d_0, d_1, ..., d_{r-1}]其中r是labels的秩，type是int32或int64，每一个entry都应该在[0, num_classes)之间
    logits=None,    # logits 是[d_0, d_1, ..., d_{r-1}, num_classes]，是float类型的，可以看成unnormalized log probabilities
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
