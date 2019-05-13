---
title: tensorflow Tensor
date: 2019-05-08 20:47:50
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.Tensor
### 目的
- 当做另一个op的输入，各个op通过Tensor连接起来，形成数据流。
- 

### 属性
- 数据类型，float32, int32, string等
- 形状

tf.Tensor一般是各种op操作后产生的变量，如tf.add,tf.log等运算，它的值是不可以改变的，没有assign()方法。

## 维度
- 0 标量
- 1 向量
- 2 矩阵
- 3 3阶张量
- n n阶张量

### 创建0维
``` python
string_scalar = tf.Variable("Elephat", tf.string)
int_scalar = tf.Variable(414, tf.int16)
float_scalar = tf.Variable(3.2345, tf.float64)
# complex_scalar = tf.Variable(12.3 - 5j, tf.complex64)
```

### 创建1维
需要列表作为初值
``` python
string_vec = tf.Variable(["Elephat"], tf.string)
int_vec = tf.Variable([414, 32], tf.int16)
float_vec = tf.Variable([3.2345, 32], tf.float64)
# complex_vec = tf.Variable([12.3 - 5j, 1 + j], tf.complex64)
```

### 创建2维
至少需要包含一行和一列
``` python
bool_mat = tf.Variable([[True], [False]], tf.bool)
string_mat = tf.Variable(["Elephat"], tf.string)
int_mat = tf.Variable([[414], [32]], tf.int16)
float_mat = tf.Variable([[3.2345, 32]], tf.float64)
# complex_mat = tf.Variable([[12.3 - 5j], [1 + j]], tf.complex64)
```

### 获取维度
``` python
tf.rank(tensor)
```

## 切片
0阶标量不需要索引，本身就是一个数字
1阶向量，可以传递一个索引访问某个数字
2阶矩阵，可以传递两个数字，返回一个标量，传递1个数字返回一个向量。
可以使用:访问，表示不操作该维度。

## 获得shape
tf.Tensor.shape

## 改变tensor的shape
### api
tf.reshape(tensor, shape, name=None)
- tensor 输入待操作tensor
- shape reshape后的shape

### 代码示例
``` python
# t = [1, 2, 3, 4, 5, 6, 7, 8, 9]
tf.reshape(t, [3, 3])  # [[1, 2, 3,], [4, 5, 6], [7, 8, 9]]
```

## 增加数据维度 
### API
tf.expand_dims(input, axis=None, name=None, dim=None)

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_expand_dims.py)
``` python
import tensorflow as tf
import numpy as np

x = tf.placeholder(tf.int32, [None, 10])
y1 = tf.expand_dims(x, 0)
y2 = tf.expand_dims(x, 1)
y3 = tf.expand_dims(x, 2)
y4 = tf.expand_dims(x, -1) # -1表示最后一维
# y5 = tf.expand_dims(x, 3) error

with tf.Session() as sess:
   inputs = np.random.rand(12, 10)
   r1, r2, r3, r4 = sess.run([y1, y2, y3, y4], feed_dict={x: inputs})
   print(r1.shape)
   print(r2.shape)
   print(r3.shape)
   print(r4.shape)
```

## 改变数据类型
### API
tf.cast(x, dtype, name=None)
- x  # 待转换数据
- dtype # 待转换数据类型

### 代码示例
``` python
x = tf.constant([1.8, 2.2], dtype=tf.float32)
tf.cast(x, tf.int32)
```

## 评估张量
tf.Tensor.eval() 返回一个与Tensor内容相同的numpy数组

### 代码示例
``` python
constant = tf.constant([1, 2, 3])
tensor = constant * constant
print(tensor.eval()) # 注意，只有eval()处于活跃的Session中才会起作用。
```

## 特殊类型
- tf.Variable 和tf.Tensor还不一样，[点击查看tf.Variable详细介绍]()
- tf.constant
- tf.placeholder
- tf.SparseTensor

### tf.placeholder
#### API
返回一个Tensor
tf.placeholder(dtype, shape=None, name = None)
- dtype  # 类型
- shape  # 形状

#### 代码示例
``` python
import tensorflow as tf
import numpy as np

x = tf.placeholder(tf.float32, shape=(None, 1024))
y = tf.matmul(x, x)

sess = tf.Session()
# print(sess.run(y))  this will fail
rand_array = np.random.rand(1024, 1024)
print(sess.run(y, feed_dict={x: rand_array}))
```

### tf.constant
#### api
tf.constant(values, dtype=None, shape=None, name='Const', verify_shape=False)
返回一个constant的Tensor。
- values # 初始值
- dtype # 类型
- shape # 形状
- name  # 可选
- verify_shape

#### 代码示例
``` python
tensor = tf.constant([1, 2, 3, 4, 5, 6])
tensor = tf.constant(-1.0, shape=[3, 4])
```


### tf.Variable
#### api
tf.Variable.\_\_init\_\_(initial_value=None, trainable=True, collections=None, validate_shape=True, caching_device=None, name=None, ...)

#### 代码示例
``` python
tensor1 = tf.Variable([[1,2], [3,5]])
tensor2 = tf.Variable(tf.constant([[1,2], [3,5]]))
sess.run(tf.global_variables_initializer())
sess.run(tensor1)
sess.run(tensor2)
```

### 创建常量Tensor
- tf.ones(shape, dtype=tf.float32, name=None)
- tf.zeros(shape, dtype=tf.float32, name=None)
- tf.fill(shape, value, name=None)
- tf.constant(value, dtype=None, shape=None, name='Const')

- tf.ones_like(tensor, dtype=None, name=None)
- tf.zeros_like(tensor, dtype=None, name=None)

- tf.linspace()


### 创建随机Tensor
- tf.random_uniform(shape, minval=0, maxval=None, dtype=tf.float32, seed=None, name=None) 
https://www.tensorflow.org/versions/r1.8/api_docs/python/tf/random_uniform
- tf.random_normal(shape, mean=0.0, stddev=1.0, dtype=tf.float32, seed=None, name=None)
均值为mean，方差为stddev的正态分布
https://www.tensorflow.org/versions/r1.8/api_docs/python/tf/random_normal
- tf.truncated_normal(shape, mean=0.0, stddev=1.0, dtype=tf.float32, seed=None, name=None)
均值为mean，方差为stddev的正态分布，保留[mean-2\*stddev, mean+2\*stddev]之内的随机数。
- tf.random_shuffle(value, seed=None, name=None)
对value的第一维重新排列

## 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_tensor.py)

``` python
import tensorflow as tf


sess = tf.Session()

x = tf.constant([[3, 4], [5, 8]])
print(sess.run(tf.constant([3,4])))
# [3 4]

print(sess.run(tf.ones_like(x)))
[[1 1]
 [1 1]]

print(sess.run(tf.zeros_like(x)))
[[0 0]
 [0 0]]

# 输出正态分布的随机采样值
print(sess.run(tf.random_normal([2,2])))
# [[-0.5188188   0.77538687]
 [ 1.2343276  -0.58534193]]

# 输出均匀[0,1]分布的随机采样值。
print(sess.run(tf.random_uniform([2,2])))
[[0.8851745  0.12824357]
 [0.28489232 0.76961493]]

print(sess.run(tf.random_uniform([2,2], dtype=tf.int32, maxval=4)))
[[0 2]
 [2 1]]

print(sess.run(tf.ones([3, 4])))
[[1. 1. 1. 1.]
 [1. 1. 1. 1.]
 [1. 1. 1. 1.]]

print(sess.run(tf.zeros([2,2])))
[[0. 0.]
 [0. 0.]]
```

## 参考文献
1.https://www.tensorflow.org/guide/tensors?hl=zh_cn
