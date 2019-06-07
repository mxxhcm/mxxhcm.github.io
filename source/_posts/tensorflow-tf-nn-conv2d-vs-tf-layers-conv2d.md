---
title: tensorflow tf.nn.conv2d vs tf.layers.conv2d
date: 2019-05-19 01:18:00
tags:
- tensorflow
- python
categories: tensorflow
---

## nn.conv2d vs layers.con2d
tf.nn.conv2d需要手动创建tensor，传入filter的参数[kernel_height, kernel_width, in_channels, num_filters]。

## 参考文献
1.https://stackoverflow.com/questions/42785026/tf-nn-conv2d-vs-tf-layers-conv2d
2.https://stackoverflow.com/a/53683545
3.https://stackoverflow.com/a/45308609
