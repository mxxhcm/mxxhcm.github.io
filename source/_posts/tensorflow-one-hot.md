---
title: tensorflow one_hot
date: 2019-07-13 12:08:55
tags:
 - tensorflow 
 - python
categories: tensorflow
---

## tf.one_hot
### 一句话介绍
返回一个one-hot tensor

### API
``` python
tf.one_hot(
    indices,    # 每一个one-hot向量不为0的维度
    depth,  # 指定每一个one-hot向量的维度
    on_value=None,  # indices上取该值
    off_value=None,     #其他地方取该值
    axis=None,
    dtype=None,
    name=None
)
```
 
### 代码示例
``` python
import tensorflow as tf

indices = [1, 1, 1]
depth = 5

result = tf.one_hot(indices, depth)

sess = tf.Session()
print(sess.run(result))
# [[0. 1. 0. 0. 0.]
#  [0. 1. 0. 0. 0.]
#  [0. 1. 0. 0. 0.]]

```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/one_hot
