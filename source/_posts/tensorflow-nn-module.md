---
title: tensorflow nn module
date: 2019-05-18 15:25:34
tags:
- tensorflow
- python
categories: tensorflow
---
	
## tf.nn
提供神经网络op。包含构建RNN cell的rnn_cell模块和一些函数。

## tf.nn.rnn_cell
rnn_cell 用于构建RNN cells
包括以下几个类：
- class BasicLSTMCell: 弃用了，使用tf.nn.rnn_cell.LSTMCell代替。
- class BasicRNNCell: 最基本的RNN cell.
- class DeviceWrapper: 保证一个RNNCell在一个特定的device运行的op.
- class DropoutWrapper: 添加droput到给定cell的的inputs和outputs的op.
- class GRUCell: GRU cell (引用文献 http://arxiv.org/abs/1406.1078).
- class LSTMCell: LSTM cell 
- class LSTMStateTuple: tupled LSTM cell
- class MultiRNNCell: 由很多个简单cells顺序组合成的RNN cell 
- class RNNCell: 表示一个RNN cell的抽象对象
- class ResidualWrapper: 确保cell的输入被添加到输出的RNNCell warpper。

## 函数

### conv2d(...)
给定一个4d输入和filter，计算2d卷积。
#### API
``` python
tf.nn.conv2d(
    input, # 输入，[batch, in_height, in_width, in_channels]
    filter, # 4d tensor, [filter_height, filter_width, in_channels, out_channles]
    strides, # 长度为4的1d tensor。
    padding, # string, 可选"SAME"或者"VALID"
    use_cudnn_on_gpu=True, #
    data_format='NHWC', #
    dilations=[1, 1, 1, 1], #
    name=None
)
```

#### 示例
``` python
def conv2d(inputs, output_dim, kernel_size, stride, initializer, activation_fn,
           padding='VALID', data_format='NHWC', name="conv2d", reuse=False):
    kernel_shape = None
    with tf.variable_scope(name, reuse=reuse):
        if data_format == 'NCHW':
            stride = [1, 1, stride[0], stride[1]]
            kernel_shape = [kernel_size[0], kernel_size[1], inputs.get_shape()[1], output_dim]
        elif data_format == 'NHWC':
            stride = [1, stride[0], stride[1], 1]
            kernel_shape = [kernel_size[0], kernel_size[1], inputs.get_shape()[-1], output_dim ]

        w = tf.get_variable('w', kernel_shape, tf.float32, initializer=initializer)
        conv = tf.nn.conv2d(inputs, w, stride, padding, data_format=data_format)

        b = tf.get_variable('b', [output_dim], tf.float32, initializer=tf.constant_initializer(0.0))
        out = tf.nn.bias_add(conv, b, data_format=data_format)

    if activation_fn is not None:
        out = activation_fn(out)
    return out, w, b
```

### convolution
#### API
``` python
tf.nn.convolution(
    input, # 输入
    filter, # 卷积核
    padding, # string, 可选"SAME"或者"VALID"
    strides=None, # 步长
    dilation_rate=None,
    name=None,
    data_format=None
)
```
#### 和tf.nn.conv2d对比
tf.nn.conv2d是2d卷积
tf.nn.convolution是nd卷积

### conv2d_transpose
反卷积
#### API
``` python
tf.nn.conv2d_transpose(
    value, # 输入，4d tensor，[batch, in_channels, height, width] for NCHW,或者[batch,height, width, in_channels] for NHWC
    filter, # 4d卷积核，shape是[height, width, output_channels, in_channels]
    output_shape, # 表示反卷积输出的shape一维tensor
    strides, # 步长
    padding='SAME',
    data_format='NHWC',
    name=None
)
```
#### 示例

### max_pool
实现max pooling
#### API
``` python
tf.nn.max_pool(
    value, # 输入，4d tensor
    ksize, # 4个整数的list或者tuple，max pooling的kernel size
    strides, # 4个整数的list或者tuple
    padding, # string, 可选"VALID"或者"VALID"
    data_format='NHWC', # string,可选"NHWC", "NCHW", NCHW_VECT_C"
    name=None
)
```

### 几个常用的函数
- bias_add(...)
- raw_rnn(...)
- static_rnn(...) # 未来将被弃用
- dynamic_rnn(...) # 未来将被弃用
- static_bidirectional_rnn(...) # 未来将被弃用
- bidirectional_dynamic_rnn(...) # 未来将被弃用
- dropout(...)
- leaky_relu(...)
- l2_loss(...)
- log_softmax(...) # 参数弃用
- softmax(...) # 参数弃用
- softmax_cross_entropy_with_logits(...)	# 未来将被弃用
- softmax_cross_entropy_with_logits_v2(...) # 参数弃用
- sparse_softmax_cross_entropy_with_logits(...)

#### 全部函数
- all_candidate_sampler(...)
- atrous_conv2d(...)
- atrous_conv2d_transpose(...)
- avg_pool(...)
- avg_pool3d(...)
- batch_norm_with_global_normalization(...)
- batch_normalization(...)
- bias_add(...)
- bidirectional_dynamic_rnn(...)
- collapse_repeated(...)
- compute_accidental_hits(...)
- conv1d(...)
- conv2d(...)
- conv2d_backprop_filter(...)
- conv2d_backprop_input(...)
- conv2d_transpose(...)
- conv3d(...)
- conv3d_backprop_filter(...)
- conv3d_backprop_filter_v2(...)
- conv3d_transpose(...)
- convolution(...) - crelu(...)
- ctc_beam_search_decoder(...)
- ctc_beam_search_decoder_v2(...)
- ctc_greedy_decoder(...)
- ctc_loss(...)
- ctc_loss_v2(...)
- ctc_unique_labels(...)
- depth_to_space(...)
- depthwise_conv2d(...)
- depthwise_conv2d_backprop_filter(...)
- depthwise_conv2d_backprop_input(...)
- depthwise_conv2d_native(...)
- depthwise_conv2d_native_backprop_filter(...)
- depthwise_conv2d_native_backprop_input(...)
- dilation2d(...)
- dropout(...)
- dynamic_rnn(...)
- elu(...)
- embedding_lookup(...)
- embedding_lookup_sparse(...)
- erosion2d(...)
- fixed_unigram_candidate_sampler(...)
- fractional_avg_pool(...)
- fractional_max_pool(...)
- fused_batch_norm(...)
- in_top_k(...)
- l2_loss(...)
- l2_normalize(...)
- leaky_relu(...)
- learned_unigram_candidate_sampler(...)
- local_response_normalization(...)
- log_poisson_loss(...)
- log_softmax(...)
- log_uniform_candidate_sampler(...)
- lrn(...)
- max_pool(...)
- max_pool3d(...)
- max_pool_with_argmax(...)
- moments(...)
- nce_loss(...)
- normalize_moments(...)
- pool(...)
- quantized_avg_pool(...)
- quantized_conv2d(...)
- quantized_max_pool(...)
- quantized_relu_x(...)
- raw_rnn(...)
- relu(...)
- relu6(...)
- relu_layer(...)
- safe_embedding_lookup_sparse(...)
- sampled_softmax_loss(...)
- selu(...)
- separable_conv2d(...)
- sigmoid(...)
- sigmoid_cross_entropy_with_logits(...)
- softmax(...)
- softmax_cross_entropy_with_logits(...)
- softmax_cross_entropy_with_logits_v2(...)
- softplus(...)
- softsign(...)
- space_to_batch(...)
- space_to_depth(...)
- sparse_softmax_cross_entropy_with_logits(...)
- static_bidirectional_rnn(...)
- static_rnn(...)
- static_state_saving_rnn(...)
- sufficient_statistics(...)
- tanh(...)
- top_k(...)
- uniform_candidate_sampler(...)
- weighted_cross_entropy_with_logits(...)
- weighted_moments(...)
- with_space_to_batch(...)
- xw_plus_b(...)
- zero_fraction(...)

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/nn
2.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell
3.https://www.tensorflow.org/api_docs/python/tf/nn/conv2d
4.https://stackoverflow.com/questions/38601452/what-is-tf-nn-max-pools-ksize-parameter-used-for
5.https://www.tensorflow.org/api_docs/python/tf/nn/convolution
6.https://stackoverflow.com/questions/47775244/difference-between-tf-nn-convolution-and-tf-nn-conv2d
7.https://www.tensorflow.org/api_docs/python/tf/nn/conv2d_transpose