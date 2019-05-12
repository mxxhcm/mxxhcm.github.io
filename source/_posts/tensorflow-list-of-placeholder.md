---
title: tensorflow list of placeholder
date: 2019-05-12 15:55:49
tags:
 - tensorflow 
 - python
categories: tensorflow
---


## feed list数据到placeholder中

### 目的
想要feed list数据到list of placeholder中去。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_placeholder_list.py)
``` python
import tensorflow as tf
import numpy as np

n = 4

ph_list = [tf.placeholder(tf.float32, [None, 10]) for _ in range(4)]
result = tf.Variable(0.0)
for x in ph_list:
    result = tf.add(result, x)
hhhh = tf.log(result)


if __name__ == "__main__":
    sess = tf.Session()
    sess.run(tf.global_variables_initializer())
    inputs = []
    for _ in range(n):
        x = np.random.rand(16, 10)
        inputs.append(x)
    # print(sess.run(hhhh, feed_dict={ph_list: inputs}))
    print(sess.run(hhhh, feed_dict={k: v for k, v in zip(ph_list, inputs)}).shape)
   
```


## 参考文献
1.https://stackoverflow.com/questions/51128427/how-to-feed-list-of-values-to-a-placeholder-list-in-tensorflow
