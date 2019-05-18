---
title: tensorflow rnn
date: 2019-05-18 15:55:34
tags:
- tensorflow
- python
categories: tensorflow
---

## 常见Cell
- class BasicRNNCell: 最基本的RNN cell.
- class LSTMCell: LSTM cell 
- class LSTMStateTuple: tupled LSTM cell
- class MultiRNNCell: 由很多个简单cells顺序组合成的RNN cell 

## BasicRNNCell
### API
``` python
__init__(
    num_units,
    activation=None,
    reuse=None,
    name=None,
    dtype=None,
    **kwargs
)
```

### 示例


### 其他
TF 2.0将会弃用，等价于tf.keras.layers.SimpleRNNCell()

## LSTMCell 
### API
``` python
__init__(
    num_units,
    use_peepholes=False,
    cell_clip=None,
    initializer=None,
    num_proj=None,
    proj_clip=None,
    num_unit_shards=None,
    num_proj_shards=None,
    forget_bias=1.0,
    state_is_tuple=True,
    activation=None,
    reuse=None,
    name=None,
    dtype=None,
    **kwargs
)
```

### 示例
### 其他
TF 2.0将会弃用，等价于tf.keras.layers.LSTMCell

## LSTMStateTuple
和LSTMCell一样，只不过state用的是tuple。
### 其他
TF 2.0将会弃用，等价于tf.keras.layers.LSTMCell

## MultiRNNCell 
相当于
### API
``` python
__init__(
    cells,
    state_is_tuple=True
)
```
### 示例
``` python
num_units = [128, 64]
cells = [BasicLSTMCell(num_units=n) for n in num_units]
stacked_rnn_cell = MultiRNNCell(cells)
```
### 其他
TF 2.0将会弃用，等价于tf.keras.layers.StackedRNNCells

## tf.nn.rnn_cell
该模块提供了许多RNN cell类和rnn函数。

### 类
- class BasicRNNCell: 最基本的RNN cell.
- class BasicLSTMCell: 弃用了，使用tf.nn.rnn_cell.LSTMCell代替，就是下面那个
- class LSTMCell: LSTM cell 
- class LSTMStateTuple: tupled LSTM cell
- class GRUCell: GRU cell (引用文献 http://arxiv.org/abs/1406.1078).
- class RNNCell: 表示一个RNN cell的抽象对象
- class MultiRNNCell: 由很多个简单cells顺序组合成的RNN cell 
- class DeviceWrapper: 保证一个RNNCell在一个特定的device运行的op.
- class DropoutWrapper: 添加droput到给定cell的的inputs和outputs的op.
- class ResidualWrapper: 确保cell的输入被添加到输出的RNNCell warpper。

## 函数
- static_rnn(...) # 未来将被弃用，和tf.contrib.rnn.static_rnn是一样的。
- dynamic_rnn(...) # 未来将被弃用
- static_bidirectional_rnn(...) # 未来将被弃用
- bidirectional_dynamic_rnn(...) # 未来将被弃用
- raw_rnn(...)

## tf.contrib.rnn
该模块提供了RNN和Attention RNN的类和函数op。

### 类
- class RNNCell: # 抽象类，所有Cell都要继承该类。所有的Warpper都要直接继承该Cell。
- class LayerRNNCell: # 所有的下列定义的Cell都要使用继承该Cell，该Cell继承RNNCell，所以所有下列Cell都间接继承RNNCell。
- class BasicRNNCell:
- class BasicLSTMCell: # 将被弃用，使用下面的LSTMCell。
- class LSTMCell:
- class LSTMStateTuple:
- class GRUCell:
- class MultiRNNCell:
- class ConvLSTMCell:
- class GLSTMCell:
- class Conv1DLSTMCell:
- class Conv2DLSTMCell:
- class Conv3DLSTMCell:
- class BidirectionalGridLSTMCell:
- class AttentionCellWrapper: 
- class CompiledWrapper:
- class CoupledInputForgetGateLSTMCell:
- class DeviceWrapper:
- class DropoutWrapper:
- class EmbeddingWrapper:
- class FusedRNNCell:
- class FusedRNNCellAdaptor:
- class GRUBlockCell:
- class GRUBlockCellV2:
- class GridLSTMCell:
- class HighwayWrapper:
- class IndRNNCell:
- class IndyGRUCell:
- class IndyLSTMCell:
- class InputProjectionWrapper:
- class IntersectionRNNCell:
- class LSTMBlockCell:
- class LSTMBlockFusedCell:
- class LSTMBlockWrapper:
- class LayerNormBasicLSTMCell:
- class NASCell:
- class OutputProjectionWrapper:
- class PhasedLSTMCell:
- class ResidualWrapper:
- class SRUCell:
- class TimeFreqLSTMCell:
- class TimeReversedFusedRNN:
- class UGRNNCell:

- ### 函数
- static_rnn(...) # 将被弃用，和tf.nn.static_rnn是一样的
- static_bidirectional_rnn(...) # 将被弃用
- best_effort_input_batch_size(...)
- stack_bidirectional_dynamic_rnn(...)
- stack_bidirectional_rnn(...)
- static_state_saving_rnn(...)
- transpose_batch_time(...)

## tf.contrib.rnn vs tf.nn.rnn_cell
事实上，这两个模块中都定义了许多RNN cell，contrib定义的是测试性的代码，而nn.rnn_cell是contrib中经过测试后的代码。
contrib中的代码会经常修改，而nn中的代码比较稳定。
contrib中的cell类型比较多，而nn中的比较少。
contrib和nn中有重复的cell，基本上nn中有的contrib中都有。

## static_rnn vs dynamic_rnn
### static_rnn
#### API
``` python
tf.nn.static_rnn(
    cell, # RNNCell的实例
    inputs, # 输入，长度为T的输入列表，列表中每一个Tensor的shape都是[batch_size, input_size]
    initial_state=None, # rnn的初始状态，如果cell.state_size是整数，它的shape需要是[batch_size, cell.state_size]，如果cell.state_size是元组，那么终究会是一个tensors的元组，[batch_size, s] for s in cell.state_size
    dtype=None,
    sequence_length=None,
    scope=None
)
```

#### 示例
``` python
  state = cell.zero_state(...)
  outputs = []
  for input_ in inputs:
    output, state = cell(input_, state)
    outputs.append(output)
  return (outputs, state)
```

### dynamic rnn
#### API
``` python
tf.nn.dynamic_rnn(
    cell, # 一个RNNCell的实例
    inputs, # RNN的输入,time_major = False, [batch_size, max_time, ...],time_major=True, [max_time, batch_size, ...]
    sequence_length=None, # 
    initial_state=None, # rnn的初始状态，如果cell.state_size是整数，它的shape需要是[batch_size, cell.state_size]，如果cell.state_size是元组，那么终究会是一个tensors的元组，[batch_size, s] for s in cell.state_size
    dtype=None,
    parallel_iterations=None,
    swap_memory=False,
    time_major=False,
    scope=None
)
```

#### 示例
``` python
# create a BasicRNNCell
rnn_cell = tf.nn.rnn_cell.BasicRNNCell(hidden_size)

# 'outputs' is a tensor of shape [batch_size, max_time, cell_state_size]

# defining initial state
initial_state = rnn_cell.zero_state(batch_size, dtype=tf.float32)

# 'state' is a tensor of shape [batch_size, cell_state_size]
outputs, state = tf.nn.dynamic_rnn(rnn_cell, input_data,
                                   initial_state=initial_state,
                                   dtype=tf.float32)

# create 2 LSTMCells
rnn_layers = [tf.nn.rnn_cell.LSTMCell(size) for size in [128, 256]]

# create a RNN cell composed sequentially of a number of RNNCells
multi_rnn_cell = tf.nn.rnn_cell.MultiRNNCell(rnn_layers)

# 'outputs' is a tensor of shape [batch_size, max_time, 256]
# 'state' is a N-tuple where N is the number of LSTMCells containing a
# tf.contrib.rnn.LSTMStateTuple for each cell
outputs, state = tf.nn.dynamic_rnn(cell=multi_rnn_cell,
                                   inputs=data,
                                   dtype=tf.float32)
```

### tf.keras.layers.RNN(cell)
在tensorflow 2.0中，上述两个API都会被弃用，使用新的keras.layers.RNN(cell)

## 参考文献 
1.https://www.tensorflow.org/api_docs/python/tf/contrib/rnn
2.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell
