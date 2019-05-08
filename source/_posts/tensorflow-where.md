---
title: tensorflow where
date: 2019-05-08 17:34:47
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.where
### 简单解释
tf.where(conditon) 返回条件为True的下标。
tf.where(condition, x=X, y=Y) 条件为True的对应位置值替换为1,为False替换成0。

### API
定义在tensorflow/python/ops/array_ops.py中。
``` python
tf.where(
    condition, # 条件
    x=None,  # 操作数1
    y=None,  # 操作数2
    name=None
)
```

### tf.where(condition)代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_where.py)
``` python
import numpy as np
import tensorflow as tf


X = tf.placeholder(tf.int32, [None, 7])

zeros = tf.zeros_like(X)
index = tf.not_equal(X, zeros)
loc = tf.where(index)

with tf.Session() as sess:
    inputs = np.array([[1, 0, 3, 5, 0, 8, 6], [2, 3, 4, 5, 6, 7, 8]])
    out = sess.run(loc, feed_dict={X: inputs})
    print(np.array(out))
    # 输出12个坐标，表示这个数组中不为0元素的索引。

```

### tf.where(condition, x=X, y=Y)代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_where.py)
``` python
import numpy as np
import tensorflow as tf


inputs = np.array([[1, 0, 3, 5, 0, 8, 6], [2, 3, 4, 5, 6, 7, 8]])
X = tf.placeholder(tf.int32, [None, 7])
zeros = tf.zeros_like(X)
ones = tf.ones_like(X)
loc = tf.where(inputs, x=ones, y=zeros)

with tf.Session() as sess:
    out = sess.run(loc, feed_dict={X: inputs})
    print(np.array(out))

```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/where 
