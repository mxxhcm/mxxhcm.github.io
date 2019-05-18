---
title: tensorflow nn
date: 2019-05-18 15:53:34
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.nn
提供神经网络op。包含构建RNN cell的rnn_cell模块和一些functions。

### tf.nn.rnn_cell
rnn_cell 用于构建RNN cells
包括以下几个类：
class BasicLSTMCell: 弃用了，使用tf.nn.rnn_cell.LSTMCell代替。
class BasicRNNCell: 最基本的RNN cell.
class DeviceWrapper: 保证一个RNNCell在一个特定的device运行的op.
class DropoutWrapper: 添加droput到给定cell的的inputs和outputs的op.
class GRUCell: GRU cell (引用文献 http://arxiv.org/abs/1406.1078).
class LSTMCell: LSTM cell 
class LSTMStateTuple: tupled LSTM cell
class MultiRNNCell: 由很多个简单cells顺序组合成的RNN cell 
class RNNCell: 表示一个RNN cell的抽象对象
class ResidualWrapper: 确保cell的输入被添加到输出的RNNCell warpper。

### 函数
#### 几个常用的函数
- bias_add(...)
- conv2d(...)
- dropout(...)
- leaky_relu(...)
- l2_loss(...)
- log_softmax(...)
- softmax(...)
- softmax_cross_entropy_with_logits(...)
- softmax_cross_entropy_with_logits_v2(...)
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
- convolution(...)
- crelu(...)
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
