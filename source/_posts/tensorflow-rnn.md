---
title: tensorflow rnn
date: 2019-05-18 15:55:34
tags:
- tensorflow
- python
categories: tensorflow
---

## 常见Cell和函数
- tf.nn.rnn_cell.BasicRNNCell: 最基本的RNN cell.
- tf.nn.rnn_cell.LSTMCell: LSTM cell 
- tf.nn.rnn_cell.LSTMStateTuple: tupled LSTM cell
- tf.nn.rnn_cell.MultiRNNCell: 多层Cell
- tf.nn.rnn_cell.DropoutCellWrapper: 给Cell加上dropout 
- tf.nn.dynamic_rnn: 动态rnn
- tf.nn.static_rnn: 静态rnn

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
[完整代码地址]()
``` python
    myrnn = rnn.BasicRNNCell(rnn_size,activation=tf.nn.relu)
    zero_state = myrnn.zero_state(batch_size, dtype=tf.float32)
    outputs, states = rnn.static_rnn(myrnn, x, initial_state=zero_state, dtype=tf.float32)
	return outputs
```

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
[完整代码地址]()    
``` python
	lstm = rnn.BasicLSTMCell(lstm_size, forget_bias=1, state_is_tuple=True)
    zero_state = lstm.zero_state(batch_size, dtype=tf.float32)
    outputs, states = rnn.static_rnn(lstm, x, initial_state=zero_state, dtype=tf.float32)
	return outputs
```
 
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
#### 代码1
``` python
num_units = [128, 64]
cells = [BasicLSTMCell(num_units=n) for n in num_units]
stacked_rnn_cell = MultiRNNCell(cells)
outputs, state = tf.nn.dynamic_rnn(cell=stacked_rnn_cell,
                                   inputs=data,
                                   dtype=tf.float32)
```
#### 代码2
[完整代码地址]()
``` python
    lstm_cell = rnn.BasicLSTMCell(lstm_size, forget_bias=1, state_is_tuple=True)
    cell = rnn.MultiRNNCell([lstm_cell]*layers, state_is_tuple=True)
    state = cell.zero_state(batch_size, dtype=tf.float32)
    outputs = []
    with tf.variable_scope("Multi_Layer_RNN", reuse=reuse):
        for time_step in range(time_steps):
            if time_step > 0:
                tf.get_variable_scope().reuse_variables()
            
            cell_outputs, state = cell(x[time_step], state)
            outputs.append(cell_outputs)
	return outputs 
```

### 其他
TF 2.0将会弃用，等价于tf.keras.layers.StackedRNNCells

## DropoutCellWrapper
### API
``` python
__init__(
    cell, # 
    input_keep_prob=1.0,
    output_keep_prob=1.0,
    state_keep_prob=1.0,
    variational_recurrent=False,
    input_size=None,
    dtype=None,
    seed=None,
    dropout_state_filter_visitor=None
)
```
### 示例
[完整代码地址]()
``` python
    lstm_cell = rnn.BasicLSTMCell(lstm_size, forget_bias=1, state_is_tuple=True)
    lstm_cell = rnn.DropoutWrapper(lstm_cell, output_keep_prob=0.9)
    cell = rnn.MultiRNNCell([lstm_cell]*layers, state_is_tuple=True)
    state = cell.zero_state(batch_size, dtype=tf.float32)
    outputs = []
    with tf.variable_scope("Multi_Layer_RNN"):
        for time_step in range(time_steps):
            if time_step > 0:
                tf.get_variable_scope().reuse_variables()
            cell_outputs, state = cell(x[time_step], state)
            outputs.append(cell_outputs)
	return outputs
```

### 其他

## static_rnn
### API
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

### 示例
``` python
	myrnn = tf.nn.rnn_cell.BasicRNNCell(rnn_size,activation=tf.nn.relu)
    zero_state = myrnn.zero_state(batch_size, dtype=tf.float32)
    outputs, states = tf.nn.static_rnn(myrnn, x, initial_state=zero_state, dtype=tf.float32)
```

## dynamic rnn
### API
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
# 最简单形式的RNN，就是该API的参数都是用默认值，给定cell和inputs，相当于做了以下操作：
#    state = cell.zero_state(...)
#    outputs = []
#    for input_ in inputs:
#      output, state = cell(input_, state)
#      outputs.append(output)
#    return (outputs, state)
```

### 示例
``` python
# 例子1.创建一个BasicRNNCell
rnn_cell = tf.nn.rnn_cell.BasicRNNCell(hidden_size)

# 定义初始化状态
initial_state = rnn_cell.zero_state(batch_size, dtype=tf.float32)

# 'outputs' shape [batch_size, max_time, cell_state_size]
# 'state' shape [batch_size, cell_state_size]
outputs, state = tf.nn.dynamic_rnn(rnn_cell, input_data,
                                   initial_state=initial_state,
                                   dtype=tf.float32)

# 例子2.创建两个LSTMCells
rnn_layers = [tf.nn.rnn_cell.LSTMCell(size) for size in [128, 256]]

# 创建一个多层RNNCelss。
multi_rnn_cell = tf.nn.rnn_cell.MultiRNNCell(rnn_layers)

# 'outputs' is a tensor of shape [batch_size, max_time, 256]
# 'state' is a N-tuple where N is the number of LSTMCells containing a
# tf.contrib.rnn.LSTMStateTuple for each cell
outputs, state = tf.nn.dynamic_rnn(cell=multi_rnn_cell,
                                   inputs=data,
                                   dtype=tf.float32)
```

## static_rnn vs dynamic_rnn

### tf.keras.layers.RNN(cell)
在tensorflow 2.0中，上述两个API都会被弃用，使用新的keras.layers.RNN(cell)



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

### 函数
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

## 参考文献 
1.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/RNNCell
2.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/BasicRNNCell
3.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/LSTMCell
4.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/MultiRNNCell
5.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/LSTMStateTuple
6.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/DropoutWrapper
7.https://www.tensorflow.org/api_docs/python/tf/nn/static_rnn
8.https://www.tensorflow.org/api_docs/python/tf/nn/dynamic_rnn
9.https://www.tensorflow.org/api_docs/python/tf/contrib/rnn
10.https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell
11.https://www.cnblogs.com/wuzhitj/p/6297992.html
12.https://stackoverflow.com/questions/48001759/what-is-right-batch-normalization-function-in-tensorflow
