---
title: tensorflow tf.nn.conv2d vs tf.layers.conv2d
date: 2019-05-19 01:18:00
tags:
- tensorflow
- python
categories: tensorflow
---

## API
### tf.layer.conv2d
``` python
tf.layers.conv2d(inputs, filters, kernel_size, strides=(1, 1), padding='valid', data_format='channels_last', dilation_rate=(1, 1), activation=None, use_bias=True, kernel_initializer=None, bias_initializer=tf.zeros_initializer(), kernel_regularizer=None, bias_regularizer=None, activity_regularizer=None, trainable=True, name=None, reuse=None)
```
### tf.nn.conv2d
``` python
tf.nn.conv2d(input, filter, strides, padding, use_cudnn_on_gpu=None, data_format=None, name=None)
```

## nn.conv2d vs layers.conv2d
tf.nn.conv2d需要手动创建filter的tensor，传入filter的参数[kernel_height, kernel_width, in_channels, num_filters]。
tf.layer.conv2d需要传入filter的维度即可。

对于tf.nn.conv2d，
filter:和input的type一样，是一个4D的tensor，shape为[filter_height, filter_width, in_channels, out_channels]
对于tf.layers.conv2d，
filters:是整数，是需要多少个filters。

可以使用tf.nn.conv2d来加载一个pretrained model，使用tf.layers.conv2d从头开始训练一个model。


## 用法
### tf.layers.conv2d
``` python
# Convolution Layer with 32 filters and a kernel size of 5
conv1 = tf.layers.conv2d(x, 32, 5, activation=tf.nn.relu) 
# Max Pooling (down-sampling) with strides of 2 and kernel size of 2
conv1 = tf.layers.max_pooling2d(conv1, 2, 2)
```

### tf.nn.conv2d
``` python
strides = 1
# Weights matrix looks like: [kernel_size(=5), kernel_size(=5), input_channels (=3), filters (= 32)]
# Similarly bias = looks like [filters (=32)]
out = tf.nn.conv2d(input, weights, padding="SAME", strides = [1, strides, strides, 1])
out = tf.nn.bias_add(out, bias)
out = tf.nn.relu(out)
```
## 参考文献
1.https://stackoverflow.com/questions/42785026/tf-nn-conv2d-vs-tf-layers-conv2d
2.https://stackoverflow.com/a/53683545
3.https://stackoverflow.com/a/45308609
