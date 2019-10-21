---
title: tensorflow contrib vs layers vs nn
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

## tf.nn.conv2d vs tf.layers.conv2d

### API
#### tf.layer.conv2d
``` python
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
    trainable=True, 
    name=None, 
    reuse=None
)
```

#### tf.nn.conv2d
``` python
tf.nn.conv2d(
    input, 
    filter, 
    strides, 
    padding, 
    use_cudnn_on_gpu=None, 
    data_format=None, 
    name=None
)
```

### nn.conv2d vs layers.conv2d
tf.nn.conv2d需要手动创建filter的tensor，传入filter的参数[kernel_height, kernel_width, in_channels, num_filters]。
tf.layer.conv2d需要传入filter的维度即可。

对于tf.nn.conv2d，
filter:和input的type一样，是一个4D的tensor，shape为[filter_height, filter_width, in_channels, out_channels]
对于tf.layers.conv2d，
filters:是整数，是需要多少个filters。

可以使用tf.nn.conv2d来加载一个pretrained model，使用tf.layers.conv2d从头开始训练一个model。


### 用法
#### tf.layers.conv2d
``` python
# Convolution Layer with 32 filters and a kernel size of 5
conv1 = tf.layers.conv2d(x, 32, 5, activation=tf.nn.relu) 
# Max Pooling (down-sampling) with strides of 2 and kernel size of 2
conv1 = tf.layers.max_pooling2d(conv1, 2, 2)
```

#### tf.nn.conv2d
``` python
strides = 1
# Weights matrix looks like: [kernel_size(=5), kernel_size(=5), input_channels (=3), filters (= 32)]
# Similarly bias = looks like [filters (=32)]
out = tf.nn.conv2d(input, weights, padding="SAME", strides = [1, strides, strides, 1])
out = tf.nn.bias_add(out, bias)
out = tf.nn.relu(out)
```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/contrib
2.https://stackoverflow.com/questions/48001759/what-is-right-batch-normalization-function-in-tensorflow
3.https://stackoverflow.com/a/48003210
4.https://stackoverflow.com/questions/42785026/tf-nn-conv2d-vs-tf-layers-conv2d
5.https://stackoverflow.com/a/53683545
6.https://stackoverflow.com/a/45308609
