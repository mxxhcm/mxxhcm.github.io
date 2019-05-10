---
title: tensorflow cond
date: 2019-05-10 17:01:14
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.cond

### API

### 代码示例1
[代码地址]()
``` python
import tensorflow as tf
import numpy as np


x = tf.placeholder(tf.int32, [10])
y = tf.constant([10, 3.2])

for i in range(10):
    y = tf.cond(tf.equal(x[i], 0), lambda: tf.add(y, 1), lambda: tf.add(y, 10))

# for i in range(10):
#     if x[i] == 0:
#         y = tf.add(y, 1)
#     else:
#         y = tf.add(y, 10)

result = tf.log(y)

with tf.Session() as sess:
   inputs = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
   print(sess.run(result, feed_dict={x: inputs}))

```

### 代码示例2
``` python
```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/cond
2.https://stackoverflow.com/questions/48571521/tensorflow-error-using-a-tf-tensor-as-a-python-bool-is-not-allowed
3.https://blog.csdn.net/Cerisier/article/details/79819248

