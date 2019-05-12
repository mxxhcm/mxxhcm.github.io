---
title: tensorflow reduction
date: 2019-05-09 11:19:00
tags:
 - tensorflow 
 - python
categories: tensorflow
---

## tf.Reduction
- tf.reduce_sum(input_tensor, reduction_indices=None, keep_dims=False, name=None)  # 计算input_tensor的和，可指定dim。
- tf.reduce_mean(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor的均值，可指定dim。
- tf.reduce_min(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor的最小值，可指定dim。
- tf.reduce_max(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor的最大值，可指定dim。
- tf.recude_proc(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor的乘积，可指定dim。
- tf.reduce_all(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor中所有元素的逻辑与，可指定dim。
- tf.reduce_any(input_tensor, reduction_indices=None, keep_dims=False, name=None) # 计算input_tensor的所有元素的逻辑或，可指定dim。
- tf.accumulate_n(inputs, shape=None, tensor_dtype=None, name=None) # 计算inputs的和。
- tf.cumsum(x, axis=0, exclusive=False, reverse=False, name=None) # 计算input_tensor的累积和。

## 代码示例
### tf.reduce_sum()
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_reduce_sum.py)
``` python
import tensorflow as tf
import numpy as np


x = tf.placeholder(dtype=tf.float32, shape=[None, 2])
y = tf.log(x)
# 对所有y求和
loss = tf.reduce_sum(y)

with tf.Session() as sess :
    # inputs = tf.constant([1.0, 2.0])
    inputs = np.array([[1.0, 2.0], [3, 4]])
    l = sess.run(loss, feed_dict={x: inputs})
    print(l)
```

## 参考文献
