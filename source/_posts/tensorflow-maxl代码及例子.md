---
title: tensorflow maxl代码及例子
date: 2019-05-08 17:38:46
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.max
### tf.maximum
比较两个tensor，返回element-wise两个tensor的最大值。
如下[示例](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_maximum.py)：
``` python
import tensorflow as tf

sess = tf.Session()
a = tf.Variable([1, 2, 3])
b = tf.Variable([2, 1, 4])

sess.run(tf.global_variables_initializer())
print("a: ", sess.run(a))
print("b: ", sess.run(b))
c = tf.maximum(a, b)

print("tf.maximum(a, b):\n  ", sess.run(c))
```
输出如下：
> a:  [1 2 3]
b:  [2 1 4]
tf.maximum(a, b):
   [2 2 4]


