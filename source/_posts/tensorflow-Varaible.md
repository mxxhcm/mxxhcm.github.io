---
title: tensorflow Varaible
date: 2019-05-12 20:41:34
tags:
 - tensorflow
 - python
categories: tensorflow
---


## 创建变量
主要有两种方式。
tf.Variable()和tf.get_variable()，他们获得的都是tensorflow.python.ops.variables.Variable类型的对象。
tf.get_variable()和tf.Variable()相比，必须提供name，以及shape，tf.Variable()不需要指定name，shape，但是需要指定初值，tf.get_varialbe()不需要初值（有默认初始化器，可以指定）。

### tf.Variable()
#### 一句话介绍
创建一个类操作全局变量。
tf.Variable 表示可通过对其运行op来改变其值的张量。与 tf.Tensor对象不同，tf.Variable 存在于单个session.run调用的上下文之外。
在TensorFlow内部，tf.Variable会存储持久性张量。具体op读取和修改此张量的值。这些修改在多个 tf.Session 之间是可见的，因此对于一个 tf.Variable，多个工作器可以看到相同的值。


#### API
``` python
tf.Variable.\_\_init\_\_(initial_value=None, trainable=True, collections=Non    e, validate_shape=True, caching_device=None, name=None, ...)
```

#### 代码示例
``` python
tensor1 = tf.Variable([[1,2], [3,5]])
tensor2 = tf.Variable(tf.constant([[1,2], [3,5]]))
sess.run(tf.global_variables_initializer())
sess.run(tensor1)
sess.run(tensor2)
```

#### 初始化
tf.Variable()生成的变量必须初始化，tf.constant()可以不用初始化。
- 使用全局初始化
sess.run(tf.global_variables_initializer())
- 使用checkpoint
- 使用tf.assign赋值

### tf.get_variable() 
#### 一句话介绍
获取一个已经存在的变量或者创建一个新的变量。主要目的，变量复用。

#### API
``` python
tf.get_variable(
    name,
    shape=None,
    dtype=None,
    initializer=None,
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
#### 代码示例
``` python
with tf.variable_scope("model") as scope:
  output1 = my_image_filter(input1)
  scope.reuse_variables()
  output2 = my_image_filter(input2)
```

## 变量集合
### 默认集合
默认情况下，每个tf.Variable()都在以下两个集合中：
- tf.GraphKeys.GLOBAL_VARIABLES - 可以在多台设备间共享的变量，
- tf.GraphKeys.TRAINABLE_VARIABLES - TensorFlow 将计算其梯度的变量。

如果不希望变量可训练，可以将其添加到 tf.GraphKeys.LOCAL_VARIABLES 集合中。以下代码将名为 my_local 的变量添加到此集合中：
``` python
my_local = tf.get_variable("my_local", shape=(), collections=[tf.GraphKeys.LOCAL_VARIABLES])
```
或者可以指定 trainable=False（作为 tf.get_variable 的参数）：
``` python
my_non_trainable = tf.get_variable("my_non_trainable",
                                   shape=(),
                                   trainable=False)
```

### 获取集合
要检索放在某个集合中的所有变量的列表，可以使用：
##### 代码示例
[代码地址]()
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

### 自定义集合
#### 添加自定义集合
可以使用自己的集合。集合名称可为任何字符串，且无需显式创建。创建变量（或任何其他对象）后调用 tf.add_to_collection将其添加到集合中。以下代码将 my_local 变量添加到名为 my_collection_name 的集合中：
``` python
tf.add_to_collection("my_collection_name", my_local)
```

## 初始化变量
### 初始化所有变量
调用 tf.global_variables_initializer()在训练开始前一次性初始化所有可训练变量。此函数会返回一个op，负责初始化 tf.GraphKeys.GLOBAL_VARIABLES 集合中的所有变量。运行此op会初始化所有变量。
``` python
sess.run(tf.global_variables_initializer())
```
### 初始化单个变量
运行变量的初始化器操作。
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

### 代码示例1
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

### 代码示例2
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
