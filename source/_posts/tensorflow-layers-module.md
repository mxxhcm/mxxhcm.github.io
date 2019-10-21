---
title: tensorflow layers
date: 2019-05-18 15:37:50
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.layers
这个模块定义在tf.contrib.layers中。主要是构建神经网络，正则化和summaries等op。它包括1个模块，19个类，以及一系列函数。

## 模块
### experimental module
tf.layers.experimental的公开的API

## 类
### class Conv2D
二维卷积类。

#### API
``` python
__init__(
    filters, # 卷积核的数量
    kernel_size, # 卷积核的大小
    strides=(1, 1),
    padding='valid',
    data_format='channels_last', # string, "channels_last", "channels_first"
    dilation_rate=(1, 1), #
    activation=None, # 激活函数
    use_bias=True,
    kernel_initializer=None, # 卷积核的构造器
    bias_initializer=tf.zeros_initializer(), # bias的构造器
    kernel_regularizer=None, #  卷积核的正则化
    bias_regularizer=None,
    activity_regularizer=None,
    kernel_constraint=None,
    bias_constraint=None,
    trainable=True, # 如果为True的话，将变量添加到TRANABLE_VARIABELS collection中
    name=None,
    **kwargs
)
```

#### 示例
#### 其他

### 所有类
- class AveragePooling1D
- class AveragePooling2D
- class AveragePooling3D
- class BatchNormalization
- class Conv1D
- class Conv2D
- class Conv2DTranspose
- class Conv3D
- class Conv3DTranspose
- class Dense
- class Dropout
- class Flatten
- class InputSpec
- class Layer
- class MaxPooling1D
- class MaxPooling2D
- class MaxPooling3D
- class SeparableConv1D
- class SeparableConv2D

 
## 函数
### conv2d
#### API
``` python
tf.layers.conv2d(
    inputs, # 输入
    filters, #  一个整数,输出的维度，就是有几个卷积核
    kernel_size,
    strides=(1, 1),
    padding='valid',
    data_format='channels_last',
    dilation_rate=(1, 1),
    activation=None,
    use_bias=True,
    kernel_initializer=None,
    bias_initializer=tf.zeros_initializer(),
    kernel_regularizer=None,
    bias_regularizer=None,
    activity_regularizer=None,
    kernel_constraint=None,
    bias_constraint=None,
    trainable=True,
    name=None,
    reuse=None
)
```
#### 示例
#### 其他


### 所有函数
需要注意的是，下列所有函数在以后版本都将被弃用。
- average_pooling1d(...)
- average_pooling2d(...)
- average_pooling3d(...)
- batch_normalization(...)
- conv1d(...)
- conv2d(...)
- conv2d_transpose(...)
- conv3d(...)
- conv3d_transpose(...)
- dense(...)
- dropout(...)
- flatten(...)
- max_pooling1d(...)
- max_pooling2d(...)
- max_pooling3d(...)
- separable_conv1d(...)
- separable_conv2d(...)

## tf.layers.conv2d vs tf.layers.Conv2d
``` python
tf.layers.Conv2d.__init__(
    filters,
    kernel_size,
    strides=(1, 1),
    padding='valid',
    data_format='channels_last',
    dilation_rate=(1, 1),
    activation=None,
    use_bias=True,
    kernel_initializer=None,
    bias_initializer=tf.zeros_initializer(),
    kernel_regularizer=None,
    bias_regularizer=None,
    activity_regularizer=None,
    kernel_constraint=None,
    bias_constraint=None,
    trainable=True,
    name=None,
    **kwargs
)
tf.layers.conv2d(
    inputs,
    filters,
    kernel_size,
    strides=(1, 1),
    padding='valid',
    data_format='channels_last',
    dilation_rate=(1, 1),
    activation=None,
    use_bias=True,
    kernel_initializer=None,
    bias_initializer=tf.zeros_initializer(),
    kernel_regularizer=None,
    bias_regularizer=None,
    activity_regularizer=None,
    kernel_constraint=None,
    bias_constraint=None,
    trainable=True,
    name=None,
    reuse=None
)

```

conv2d是函数；Conv2d是类。
conv2d运行的时候需要传入卷积核参数，输入；Conv2d在构造的时候需要实例化卷积核参数，实例化后，可以使用不用的输入得到不同的输出。
调用conv2d就相当于调用Conv2d对象的apply(inputs)函数。

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/layers
4.https://www.tensorflow.org/api_docs/python/tf/layers/Conv2D
5.https://www.tensorflow.org/api_docs/python/tf/layers/conv2d
6.https://stackoverflow.com/questions/52011509/what-is-difference-between-tf-layers-conv2d-and-tf-layers-conv2d/52035621
