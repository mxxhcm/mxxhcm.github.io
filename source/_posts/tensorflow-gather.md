---
title: tensorflow gather
date: 2019-05-11 21:03:00
tags:
 - tensorflow
categories: tensorflow
mathjax: true
---


## tf.gather_nd
### 一句话介绍
按照索引将输入tensor的某些维度拼凑成一个新的tenosr
### API
``` python
tf.gather_nd(
    params, # 输入参数
    indices, # 索引
    name=None #
)
```
indices是一个K维的整形tensor。
indices的最后一维至多和params的rank一样大，如果indices.shape==params.rank，那么对应的是elements，如果indices.shape $\lt$ params.rank，那么对应的是slices。输出的tensor shape是：
indices.shape[:-1] + params.shape[indices.shape[-1]:]
原文如下：
> The last dimension of indices corresponds to elements (if indices.shape[-1] == params.rank) or slices (if indices.shape[-1] < params.rank) along dimension indices.shape[-1] of params. The output tensor has shape
indices.shape[:-1] + params.shape[indices.shape[-1]:]

如果indices是两维的，那么就相当于用第二维的indices去访问params，然后indices的第一维度相当于把第二维的tensor放入一个列表。
indices是高维（大于两维）的话，反正就是找最后一维的维度，然后到params中找对应的数。
``` python
    indices = [[[1]], [[0]]]
    params = [[['a0', 'b0'], ['c0', 'd0']],
              [['a1', 'b1'], ['c1', 'd1']]]
    output = [[[['a1', 'b1'], ['c1', 'd1']]],
              [[['a0', 'b0'], ['c0', 'd0']]]]
# 直接看indices的最后一维，然后到params中找，比如[1]，找params[1]=[['a1', 'b1'], ['c1', 'd1']]],params[0]=[['a0', 'b0'], ['c0', 'd0']]。然后在组成output，shape怎么确定？我的理解是，直接用params[1]的结果去替换indices中的[1]，也就是[[params[1]]]

    indices = [[[0, 1], [1, 0]], [[0, 0], [1, 1]]]
    params = [[['a0', 'b0'], ['c0', 'd0']],
              [['a1', 'b1'], ['c1', 'd1']]]
    output = [[['c0', 'd0'], ['a1', 'b1']],
              [['a0', 'b0'], ['c1', 'd1']]]


    indices = [[[0, 0, 1], [1, 0, 1]], [[0, 1, 1], [1, 1, 0]]]
    params = [[['a0', 'b0'], ['c0', 'd0']],
              [['a1', 'b1'], ['c1', 'd1']]]
    output = [['b0', 'b1'], ['d0', 'c1']]
```

### 代码示例1
[代码地址]()
``` python
import tensorflow as tf
import numpy as np

sess = tf.Session()

data = np.array([[0, 1, 2, 3, 4, 5],
          [6, 7, 8, 9, 10, 11],
          [12, 13, 14, 15, 16, 17],
          [18, 19, 20, 21, 22, 23],
          [24, 25, 26, 27, 28, 29]])
data = np.reshape(np.arange(30), [5, 6])
x = tf.constant(data)
print(sess.run(x))
# [[ 0  1  2  3  4  5]
#  [ 6  7  8  9 10 11]
#  [12 13 14 15 16 17]
#  [18 19 20 21 22 23]
#  [24 25 26 27 28 29]]

# Collecting elements from a tensor of rank 2
result = tf.gather_nd(x, [1, 2])
print(sess.run(result))
# indices.shape=(2,), indices.shape[:-1]=(), indices.shape[-1]=2, params.shape=(5,6), params.shape[indices.shape[-1]:]=(), outputs.shape=()+() = () 
# 8 
result = tf.gather_nd(x, [[1, 2], [2,3]])
print(sess.run(result))
# indices.shape=(2,2), indices.shape[:-1]=(2,), indices.shape[-1]=2, params.shape=(5,6), params.shape[indices.shape[-1]:]=(), outputs.shape=(2,)+() = (2,) 
# [8, 15]
# Collecting rows from a tensor of rank 2
result = tf.gather_nd(x, [[1],[2]])
print(sess.run(result))
# indices.shape=(2, 1), indices.shape[:-1]=(2,), indices.shape[-1]=1, params.shape=(5,6), params.shape[indices.shape[-1]:]=(6,), outputs.shape=(2,)+(6,) = (2,6,) 
# [[ 6  7  8  9 10 11]
#  [12 13 14 15 16 17]]
 
```
### 代码示例2
``` python
import tensorflow as tf
import numpy as np

sess = tf.Session()

data = np.array([[[0, 1],
          [2, 3],
          [4, 5]],
         [[6, 7],
          [8, 9],
          [10,11]]])
data = np.reshape(np.arange(12), [2, 3, 2])
x = tf.constant(data)
print(sess.run(x))
#[[[ 0  1]
#  [ 2  3]
#  [ 4  5]]
# [[ 6  7]
#  [ 8  9]
#  [10 11]]]

# Collecting elements from a tensor of rank 3
result = tf.gather_nd(x, [[0, 0, 0], [1, 2, 1]])
print(sess.run(result))
# indices.shape=(2, 3), indices.shape[:-1]=(2,), indices.shape[-1]=3, params.shape=(2, 3, 2), params.shape[indices.shape[-1]:]=(), outputs.shape=(2,)+() = (2,) 
# [0 11]

# Collecting batched rows from a tensor of rank 3
result = tf.gather_nd(x, [[[0, 0], [0, 1]], [[1, 0], [1, 1]]])
print(sess.run(result))
# indices.shape=(2, 2, 2), indices.shape[:-1]=(2, 2, ), indices.shape[-1]=2, params.shape=(2, 3, 2), params.shape[indices.shape[-1]:]=(2,), outputs.shape=(2, 2)+(2, ) = (2, 2, 2) 
# [[[0 1]
#  [2 3]]
#
# [[6 7]
#  [8 9]]]

result = tf.gather_nd(x, [[0, 0], [0, 1], [1, 0], [1, 1]])
print(sess.run(result))
# indices.shape=(4, 2), indices.shape[:-1]=(4,), indices.shape[-1]=2, params.shape=(2, 3, 2), params.shape[indices.shape[-1]:]=(2,), outputs.shape=(4,)+(2,) = (4, 2) 
# [[0 1]
#  [2 3]
#  [6 7]
#  [8 9]]

```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/gather_nd
