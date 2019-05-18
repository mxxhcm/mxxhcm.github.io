---
title: tensorflow rnn
date: 2019-05-18 15:25:34
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.nn.rnn_cell
该模块提供了RNN cell类的op。
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

## tf.contrib.rnn

### 类
该模块提供了RNN和Attention RNN的类和函数op。
- class AttentionCellWrapper
- class BasicLSTMCell
- class BasicRNNCell
- class BidirectionalGridLSTMCell
- class CompiledWrapper
- class Conv1DLSTMCell
- class Conv2DLSTMCell
- class Conv3DLSTMCell
- class ConvLSTMCell
- class CoupledInputForgetGateLSTMCell
- class DeviceWrapper
- class DropoutWrapper
- class EmbeddingWrapper
- class FusedRNNCell
- class FusedRNNCellAdaptor
- class GLSTMCell
- class GRUBlockCell
- class GRUBlockCellV2
- class GRUCell
- class GridLSTMCell
- class HighwayWrapper
- class IndRNNCell
- class IndyGRUCell
- class IndyLSTMCell
- class InputProjectionWrapper
- class IntersectionRNNCell
- class LSTMBlockCell
- class LSTMBlockFusedCell
- class LSTMBlockWrapper
- class LSTMCell
- class LSTMStateTuple
- class LayerNormBasicLSTMCell
- class LayerRNNCell
- class MultiRNNCell
- class NASCell
- class OutputProjectionWrapper
- class PhasedLSTMCell
- class RNNCell
- class ResidualWrapper
- class SRUCell
- class TimeFreqLSTMCell
- class TimeReversedFusedRNN
- class UGRNNCell
- 
- ### 函数
- best_effort_input_batch_size(...)
- stack_bidirectional_dynamic_rnn(...)
- stack_bidirectional_rnn(...)
- static_bidirectional_rnn(...)
- static_rnn(...)
- static_state_saving_rnn(...)
- transpose_batch_time(...)

## 参考文献 
1.https://www.tensorflow.org/api_docs/python/tf/contrib/rnn
2.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell
