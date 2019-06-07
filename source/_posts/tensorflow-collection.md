---
title: tensorflow collection
date: 2019-05-13 10:28:29
tags:
 - tensorflow
 - python
categories: tensorflow
mathjax: true
---

## tf.collection
Tensorflow用graph collection来管理不同类型的对象。tf.GraphKeys中定义了默认的collection，tf通过调用各种各样的collection操作graph中的变量。比如tf.Optimizer只优化tf.GraphKeys.TRAINABLE_VARIABLES collection中的变量。常见的collection如下：
- GLOBAL_VARIABLES: 所有的Variable对象在创建的时候自动加入该colllection，且在分布式环境中共享（model variables是它的子集）。一般来说，TRAINABLE_VARIABLES包含在MODEL_VARIABLES中，MODEL_VARIABLES包含在GLOBAL_VARIABLES中。也就是说TRAINABLE_VARIABLES$\le$MODEL_VARIABLES$\le$GLOBAL_VARIABLES。
- LOCAL_VARIABLES: 它是GLOBAL_VARIABLES不同的是在本机器上的Variable子集。使用tf.contrib.framework.local_variable将变量添加到这个collection.
- MODEL_VARIABLES: 模型变量，在构建模型中，所有用于前向传播的Variable都将添加到这里。使用 tf.contrib.framework.model_variable向这个collection添加变量。
- TRAINALBEL_VARIABLES: 所有用于反向传播的Variable，可以被optimizer训练，进行参数更新的变量。tf.Variable对象同样会自动加入这个collection。
- SUMMARIES: graph创建的所有summary Tensor都会记录在这里面。
- QUEUE_RUNNERS: 
- MOVING_AVERAGE_VARIABLES: 保持Movering average的变量子集。
- REGULARIZATION_LOSSES: 创建graph的regularization loss。

这里主要介绍三类collection，一种是GLOBAL_VARIABLES，一种是SUMMARIES，一种是自定义的collections。

下面的一些collection也被定义了，但是并不会自动添加
> The following standard keys are defined, but their collections are not automatically populated as many of the others are:

- WEIGHTS
- BIASES
- ACTIVATIONS

## GLOBAL_Variable collection
tf.Variable()对象在生成时会被默认添加到tf.GraphKeys中的GLOBAL_VARIABLES和TRAINABLE_VARIABLES collection中。
### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_global_trainable_variables_collections.py)
``` python
import tensorflow as tf

a = tf.Variable([1, 2, 3])
b = tf.get_variable("bbb", shape=[2,3])
tf.constant([3])
c = tf.ones([3])
d = tf.random_uniform([3, 4])
e = tf.log(c)

# 查看GLOBAL_VARIABLES collection中的变量
global_variables = tf.get_collection(tf.GraphKeys.GLOBAL_VARIABLES)
for var in global_variables:
   print(var)

# 查看TRAINABLE_VARIABLES collection中的变量
trainable_variables = tf.get_collection(tf.GraphKeys.TRAINABLE_VARIABLES)
for var in global_variables:
   print(var)
```

## Summary collection
Summary op产生的变量会被添加到tf.GraphKeys.SUMMARIES collection中。
[点击查看关于tf.summary的详细介绍](https://mxxhcm.github.io/2019/05/08/tensorflow-summary/)

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_summary_collection.py)
``` python
import tensorflow as tf

# 生成一个图
graph = tf.Graph()

with graph.as_default():
    # 指定模型参数
    w = tf.Variable([0.3], name="w", dtype=tf.float32)
    b = tf.Variable([0.2], name="b", dtype=tf.float32)

    # 输入数据placeholder
    x = tf.placeholder(tf.float32, name="inputs")
    y = tf.placeholder(tf.float32, name="outputs")

    # 前向传播
    with tf.name_scope('linear_model'):
        linear = w * x + b

	# 计算loss
    with tf.name_scope('cal_loss'):
        loss = tf.reduce_mean(input_tensor=tf.square(y - linear), name='loss')

	# 定义summary saclar op
    with tf.name_scope('add_summary'):
        summary_loss = tf.summary.scalar('MSE', loss)
        summary_b = tf.summary.scalar('b', b[0])

	# 定义优化器
    with tf.name_scope('train_model'):
        optimizer = tf.train.GradientDescentOptimizer(0.01)
        train = optimizer.minimize(loss)

with tf.Session(graph=graph) as sess:
	inputs = [1, 2, 3, 4]
	outputs = [2, 3, 4, 5]
    # 定义写入文件类
    writer = tf.summary.FileWriter("./summary/", graph)
    # 获取所有的summary op，不用一个一个去单独run
    merged = tf.summary.merge_all()

	# 初始化
    init_op = tf.global_variables_initializer()
    sess.run(init_op)
    for i in range(5000):
		# 运行summary op merged
        _, summ = sess.run([train, merged], feed_dict={x: inputs, y: outputs})
		# 将summary op返回的变量转化为事件，写入文件
        writer.add_summary(summ, global_step=i)

    w_, b_, l_ = sess.run([w, b, loss], feed_dict={x: inputs, y: outputs})
    print("w: ", w_, "b: ", b_, "loss: ", l_)

    # 查看SUMMARIES collection
    for var in tf.get_collection(tf.GraphKeys.SUMMARIES):
        print(var)

```

## 自定义collection
通过tf.add_collection()和tf.get_collection()可以添加和访问custom collection。
### 示例代码
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_custom_collection.py)
``` python
import tensorflow as tf

# 定义第1个loss
x1 = tf.constant(1.0)
l1 = tf.nn.l2_loss(x1)

# 定义第2个loss
x2 = tf.constant([2.5, -0.3])
l2 = tf.nn.l2_loss(x2)

# 将loss添加到losses collection中
tf.add_to_collection("losses", l1)
tf.add_to_collection("losses", l2)

# 查看losses collection中的内容
losses = tf.get_collection('losses')
for var in tf.get_collection('losses'):
    print(var)

# 建立session运行
with tf.Session() as sess:
    init = tf.global_variables_initializer()
    sess.run(init)
    losses_val = sess.run(losses)
    print(losses_val)
```

## 疑问
collection是和graph绑定在一起的，那么如果定义了很多个图，如何获得非默认图的tf.GraphKeys中定义的collection？？

## 参考文献
1.https://blog.csdn.net/shenxiaolu1984/article/details/52815641
2.https://blog.csdn.net/hustqb/article/details/80398934
3.https://www.tensorflow.org/api_docs/python/tf/GraphKeys?hl=zh_cn
