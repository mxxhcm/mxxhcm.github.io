---
title: tensorflow tf.nn.conv2d vs tf.layers.conv2d
date: 2019-05-19 01:18:00
tags:
- tensorflow
- python
categories: tensorflow
---


All of these other replies talk about how the parameters are different, but actually, the main difference of tf.nn and tf.layers conv2d is that for tf.nn, you need to create your own filter tensor and pass it in. This filter needs to have the size of: [kernel_height, kernel_width, in_channels, num_filters]


## 参考文献
1.https://stackoverflow.com/questions/42785026/tf-nn-conv2d-vs-tf-layers-conv2d
2.https://stackoverflow.com/a/53683545
3.https://stackoverflow.com/a/45308609
