---
title: tensorflow contrib layers nn
date: 2019-05-18 15:58:59
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.contrib
根据tensorflow官网的说法，tf.contrib模块中包含了易修改的测试代码，
> contrib module containing volatile or experimental code.

当其中的某一个模块完成的时候，就会从contrib模块中移除。为了保持对历史版本的兼容性，可能这几个模块会存在同一个函数的不同实现。

## tf.nn,tf.layers和tf.contrib
tf.nn中是low-level的op
tf.layers是high-level的op
而tf.contrib中的是非正式版本的实现，在后续版本中可能会被弃用。

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/contrib
2.https://stackoverflow.com/questions/48001759/what-is-right-batch-normalization-function-in-tensorflow
3.https://stackoverflow.com/a/48003210

