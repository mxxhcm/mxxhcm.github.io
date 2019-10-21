---
title: tensorflow boolean_mask
date: 2019-05-08 17:46:26
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.boolean_mask
### 简单解释
用一个mask数组和输入的tensor做与操作，忽略为0的值。
### api
定义在tensorflow/python/ops/array_ops.py
``` python
tf.boolean_mask(
    tensor, # 要处理的tensor
    mask, # 掩码，也需要是一个tensor
    name='boolean_mask', # 这个op的名字
    axis=None #
)
```

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_boolean_mask.py)
```
import tensorflow as tf

sess = tf.Session()
a = tf.Variable([1, 2, 3])
b = tf.Variable([2, 1.0, 4.0])
c = tf.Variable([2, 1.0, 0.0])
d = tf.Variable([2, 0.0, 4.0])
e = tf.Variable([0, 1.0, 4.0])
f = tf.Variable([0, 1.0, 0.0])
g = tf.Variable([0, 0.0, 0.0])

sess.run(tf.global_variables_initializer())
print("a: ", sess.run(a))
print("b: ", sess.run(b))
print("c: ", sess.run(c))
print("d: ", sess.run(d))
print("e: ", sess.run(e))
print("f: ", sess.run(f))
print("g: ", sess.run(g))
# c = tf.maximum(a, b)
a1 = tf.boolean_mask(a, b)
a2 = tf.boolean_mask(a, c)
a3 = tf.boolean_mask(a, d)
a4 = tf.boolean_mask(a, e)
a5 = tf.boolean_mask(a, f)
a6 = tf.boolean_mask(a, g)

print("tf.boolean(a, b):\n  ", sess.run(a1))
print("tf.boolean(a, c):\n  ", sess.run(a2))
print("tf.boolean(a, d):\n  ", sess.run(a3))
print("tf.boolean(a, e):\n  ", sess.run(a4))
print("tf.boolean(a, f):\n  ", sess.run(a5))
print("tf.boolean(a, g):\n  ", sess.run(a6))
```
输出如下：
> a:  [1 2 3]
b:  [2. 1. 4.]
c:  [2. 1. 0.]
d:  [2. 0. 4.]
e:  [0. 1. 4.]
f:  [0. 1. 0.]
g:  [0. 0. 0.]
tf.boolean(a, b):
   [1 2 3]
tf.boolean(a, c):
   [1 2]
tf.boolean(a, d):
   [1 3]
tf.boolean(a, e):
   [2 3]
tf.boolean(a, f):
   [2]
tf.boolean(a, g):
   []
 

## 参考文献
1.http://landcareweb.com/questions/27920/zai-tensorflowzhong-ru-he-cong-pythonde-zhang-liang-zhong-huo-qu-fei-ling-zhi-ji-qi-suo-yin
2.https://www.tensorflow.org/api_docs/python/tf/boolean_mask
