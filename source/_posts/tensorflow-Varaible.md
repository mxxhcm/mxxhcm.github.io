---
title: tensorflow Varaible
date: 2019-05-12 20:41:34
tags:
 - tensorflow
 - python
categories: tensorflow
---


## 创建Variable
Tensorflow有两种方式创建Variable：tf.Variable()和tf.get_variable()，这两种方式获得的都是tensorflow.python.ops.variables.Variable类型的对象，但是他们的输入参数还有些不一样。
||tf.Variable()|tf.get_variable()|
|:-:|:-:|:-:|
|name|不需要|需要|
|shape|不需要|需要|
|初值|需要|不需要|

两种方法事实上都可以指定name和初值。而tf.Variable()的初值中已经包含了shape，所以不需要再显示传入shape了。这里的需要和不需要指的是必要不必要，如果没有传入需要的参数，就会报错，不需要的参数则不会影响。

## tf.Variable()
### 一句话介绍
创建一个类操作全局变量。在TensorFlow内部，tf.Variable会存储持久性张量，允许各种op读取和修改它的值。这些修改在多个Session之间是可见的，因此对于一个tf.Variable，多个工作器可以看到相同的值。
### 和tf.Tensor对比
tf.Variable 表示可通过对其运行op来改变其值的张量。与 tf.Tensor对象不同，tf.Variable 存在于单个session.run调用的上下文之外。tf.Tensor的值是不可以改变的，tf.Tensor没有assign函数。

### API
``` python
tf.Variable.__init__(
	initial_value=None,  # 指定变量的初值
	trainable=True,  # 是否在BP时训练该参数
	collections=None, # 指定变量的collection
	validate_shape=True, 
	caching_device=None, 
	name=None,  # 指定变量的名字
	...
)
```

### 代码示例
``` python
tensor1 = tf.Variable([[1,2], [3,5]])
tensor2 = tf.Variable(tf.constant([[1,2], [3,5]]))
sess.run(tf.global_variables_initializer())
sess.run(tensor1)
sess.run(tensor2)
```

### 初始化
tf.Variable()生成的变量必须初始化，tf.constant()可以不用初始化。
- 使用全局初始化
sess.run(tf.global_variables_initializer())
- 使用checkpoint
- 使用tf.assign赋值

## tf.get_variable() 
### 一句话介绍
获取一个已经存在的变量或者创建一个新的变量。主要目的，变量复用。

### API
``` python
tf.get_variable(
    name, # 指定变量的名字，必选项
    shape=None, # 指定变量的shape，可选项
    dtype=None, # 指定变量类型
    initializer=None, # 指定变量初始化器
    regularizer=None,
    trainable=None,
    collections=None,
    caching_device=None,
    partitioner=None,
    validate_shape=True,
    use_resource=None,
    custom_getter=None,
    constraint=None,
    synchronization=tf.VariableSynchronization.AUTO,
    aggregation=tf.VariableAggregation.NONE
)
```
### 代码示例
``` python
with tf.variable_scope("model") as scope:
  output1 = my_image_filter(input1)
  scope.reuse_variables()
  output2 = my_image_filter(input2)
```

## Variable和collection
[点击查看关于collecion的详细介绍](https://mxxhcm.github.io/2019/05/13/tensorflow-collection/)
默认情况下，每个tf.Variable()都在以下两个collection中：
- tf.GraphKeys.GLOBAL_VARIABLES - 可以在多台设备间共享的变量，
- tf.GraphKeys.TRAINABLE_VARIABLES - TensorFlow 将计算其梯度的变量。

如果不希望变量是可训练的，可以在创建时指定其collection为 tf.GraphKeys.LOCAL_VARIABLES collection中。
``` python
my_local = tf.get_variable("my_local", shape=(), collections=[tf.GraphKeys.LOCAL_VARIABLES])
```
或者可以指定 trainable=False：
``` python
my_non_trainable = tf.get_variable("my_non_trainable",
                                   shape=(),
                                   trainable=False)
```

### 获取collection
要检索放在某个collection中的所有变量的列表，可以使用：
#### 代码示例
[代码地址](https://github.com/mxxhcm/code/tree/master/tf/ops/tf_Variable_collection.py)
``` python
import tensorflow as tf

a = tf.Variable([1, 2, 3])
b = tf.get_variable("bbb", shape=[2,3])
tf.constant([3])
c = tf.ones([3])
d = tf.random_uniform([3, 4])
print(tf.get_collection(tf.GraphKeys.GLOBAL_VARIABLES))
# [<tf.Variable 'Variable:0' shape=(3,) dtype=int32_ref>, <tf.Variable 'bbb:0' shape=(2, 3) dtype=float32_ref>]
# 可以看出来，只有tf.Variable()和tf.get_variable()产生的变量会加入到这个图中
```

### 自定义collection
#### 添加自定义collection
可以使用自定义的collection。collection名称可为任何字符串，且无需显式创建。创建对象（包括Variable和其他）后调用 tf.add_to_collection将其添加到相应collection中。以下代码将 my_local 变量添加到名为 my_collection_name 的collection中：
``` python
tf.add_to_collection("my_collection_name", my_local)
```

## 初始化变量
### 初始化所有变量
调用 tf.global_variables_initializer()在训练开始前一次性初始化所有可训练变量。此函数会返回一个op，负责初始化 tf.GraphKeys.GLOBAL_VARIABLES collection中的所有变量。运行此op会初始化所有变量。
``` python
sess.run(tf.global_variables_initializer())
```
### 初始化单个变量
运行变量的初始化器op。
``` python
sess.run(my_variable.initializer)
```

### 查询未初始化变量
``` python
print(sess.run(tf.report_uninitialized_variables()))
```


## 共享变量
TensorFlow 支持两种共享变量的方式：
- 显式传递 tf.Variable 对象。
- 将 tf.Variable 对象隐式封装在 tf.variable_scope 对象内。

### variable_scope
#### 代码示例1
使用variable_scope区分weights和biases。
``` python
def conv_relu(input, kernel_shape, bias_shape):
    # Create variable named "weights".
    weights = tf.get_variable("weights", kernel_shape,
        initializer=tf.random_normal_initializer())
    # Create variable named "biases".
    biases = tf.get_variable("biases", bias_shape,
        initializer=tf.constant_initializer(0.0))
    conv = tf.nn.conv2d(input, weights,
        strides=[1, 1, 1, 1], padding='SAME')
    return tf.nn.relu(conv + biases)
```

#### 代码示例2
使用variable_scope声明不同作用域
``` python
def my_image_filter(input_images):
    with tf.variable_scope("conv1"):
        # Variables created here will be named "conv1/weights", "conv1/biases".
        relu1 = conv_relu(input_images, [5, 5, 32, 32], [32])
    with tf.variable_scope("conv2"):
        # Variables created here will be named "conv2/weights", "conv2/biases".
        return conv_relu(relu1, [5, 5, 32, 32], [32])
```

### 共享方式1
设置reuse=True
``` python
with tf.variable_scope("model"):
  output1 = my_image_filter(input1)
with tf.variable_scope("model", reuse=True):
  output2 = my_image_filter(input2)
```

### 共享方式2
调用scope.reuse_variables触发重用
``` python
with tf.variable_scope("model") as scope:
  output1 = my_image_filter(input1)
  scope.reuse_variables()
  output2 = my_image_filter(input2)
```

## 参考文献
1.https://blog.csdn.net/MrR1ght/article/details/81228087
2.https://www.tensorflow.org/guide/variables?hl=zh_cn
3.https://www.tensorflow.org/api_docs/python/tf/get_variable?hl=zh_cn
