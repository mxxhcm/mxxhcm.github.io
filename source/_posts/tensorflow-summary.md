---
title: tensorflow summary
date: 2019-05-08 17:39:43
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.summary
### 目的
该模块定义在tensorflow/\_api/v1/summary/\_\_init\_\_.py文件中，主要用于可视化。
每次运行完一个op之后，调用writer.add_summary()将其写入事件file。因为summary操作实在数据流的外面进行操作的，并不会操作数据，所以需要每次运行完之后，都调用一次写入函数。

## 常用API
### 函数
``` python
# 用来定义一个summary scalar op，同时会将这个op加入到tf.GraphKeys.SUMMARIES collection中。
tf.summary.scalar(
	name, 
	tensor, # 一个实数型的Tensor，包含单个的值。
	collections=None, # 可选项，是graph collections keys的list，新的summary op会被添加到这个list of collection。默认的list是[GraphKeys.SUMMARIES]。
	family=None
)
# 定义一个summary histogram op，同时会将这个op加入到tf.GraphKeys.SUMMARIES collection中。
tf.summary.histogram(
    name,
    values, # 一个实数型的Tensor，任意shape，用来生成直方图。
    collections=None, # 可选项，是graph collections keys的list，新的summary op会被添加到这个list of collection。默认的list是[GraphKeys.SUMMARIES].
    family=None
)
# 将所有定义的summary op集中到一块，如scalar，text，histogram等。
tf.summary.merge_all(  
	key=tf.GraphKeys.SUMMARIES, #指定用哪个GraphKey来collect summaries。默认设置为GraphKeys.SUMMARIES.并不是说将他们加入到哪个GraphKey的意思，tf.summary.scalar()等会将op加入到相应的colleection。
    scope=None, #
    name=None
) 
```

### scalar和histogram的区别
scalar记录的是一个标量。
而histogram记录的是一个分布，可以是任何shape。

### 函数示例
``` python
summary_loss = tf.summary.scalar('loss', loss)
summary_weights = tf.summary.scalar('weights', weights)
# merged可以代替sumary_loss和summary_weights op。
merged = tf.summary.merge_all() 
```
关于tf.summary.histogram()的示例，[可以点击查看。](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_summary_histogram.py)

### 类
``` python
# 定义将Summary数据写入event文件的类
tf.summary.FileWriter(
	self, 
	logdir,　
	graph=None, 
	max_queue=10,
	flush_secs=120, 
	graph_def=None, 
	filename_suffix=None
)
```

#### 类内函数
``` python
# 将summary op的输出存到event文件(Adds a Summary protocol buffer to the event file.)
tf.summary.FileWriter.add_summary(
	self,
	summary,  # 一个Summary protocol buffer，一般是sess.run(summary_op)的结果
	global_step=None
) 
```
### 类示例
``` python
writer = tf.summary.FileWriter("./summary/")
with tf.Session() as sess:
    summ = sess.run([merged], feed_dict={x: inputs, y: outputs})
    writer.add_summary(summ, global_step=i)
```

## 使用流程：
1. summary_op = tf.summary_scalar() # 声明summary op，会将该op变量加入tf.GraphKeys.SUMMARIES collection
2. merged = tf.summary.merge_all() # 将所有summary op合并
3. writer = tf.summary.FileWriter() # 声明一个FileWrite文件，用于将Summary数据写入event文件
4. output = sess.run([merged]) # 运行merge后的summary op
5. writer.add_summary(output) # 将op运行后的结果写入事件文件

## 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_summary.py)
``` python
graph = tf.Graph()

with graph.as_default():
    # model parameters
    w = tf.Variable([0.3], name="w", dtype=tf.float32)
    b = tf.Variable([0.2], name="b", dtype=tf.float32)

    x = tf.placeholder(tf.float32, name="inputs")
    y = tf.placeholder(tf.float32, name="outputs")

    with tf.name_scope('linear_model'):
        linear = w * x + b

    with tf.name_scope('cal_loss'):
        loss = tf.reduce_mean(input_tensor=tf.square(y - linear), name='loss')

    with tf.name_scope('add_summary'):
        summary_loss = tf.summary.scalar('MSE', loss)
        summary_b = tf.summary.scalar('b', b[0])

    with tf.name_scope('train_model'):
        optimizer = tf.train.GradientDescentOptimizer(0.01)
        train = optimizer.minimize(loss)

inputs = [1, 2, 3, 4]
outputs = [2, 3, 4, 5]

with tf.Session(graph=graph) as sess:
    writer = tf.summary.FileWriter("./summary/", graph)
    merged = tf.summary.merge_all()

    init_op = tf.global_variables_initializer()
    sess.run(init_op)
    for i in range(5000):
        _, summ = sess.run([train, merged], feed_dict={x: inputs, y: outputs})
        writer.add_summary(summ, global_step=i)

    w_, b_, l_ = sess.run([w, b, loss], feed_dict={x: inputs, y: outputs})
    print("w: ", w_, "b: ", b_, "loss: ", l_)
    for var in tf.get_collection(tf.GraphKeys.SUMMARIES):
    #for var in tf.get_collection(tf.GraphKeys.MODEL_VARIABLES):
        print(var)
```

使用tensorboard --logdir ./summary/打开tensorboard

### 官网示例
加了一定注释，[可以点击查看](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_summary_example.py)

## 所有API
### 类
- class Event: A ProtocolMessage
- class FileWriter: Writes Summary protocol buffers to event files.
- class FileWriterCache: Cache for file writers.
- class SessionLog: A ProtocolMessage
- class Summary: A ProtocolMessage
- class SummaryDescription: A ProtocolMessage
- class TaggedRunMetadata: A ProtocolMessage

### 函数
- scalar(...): Outputs a Summary protocol buffer containing a single scalar value.
- histogram(...): Outputs a Summary protocol buffer with a histogram.
- image(...): Outputs a Summary protocol buffer with images.
- tensor_summary(...): Outputs a Summary protocol buffer with a serialized tensor.proto.
- audio(...): Outputs a Summary protocol buffer with audio.
- text(...): Summarizes textual data.
- merge(...): Merges summaries.
- merge_all(...): Merges all summaries collected in the default graph.
- get_summary_description(...): Given a TensorSummary node_def, retrieve its SummaryDescription.


## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/summary
2.https://www.tensorflow.org/api_docs/python/tf/summary/scalar
3.https://www.tensorflow.org/api_docs/python/tf/summary/histogram
4.https://www.tensorflow.org/api_docs/python/tf/summary/merge_all
5.https://www.tensorflow.org/guide/graphs#visualizing_your_graph
6.https://www.tensorflow.org/guide/summaries_and_tensorboard
7.https://www.tensorflow.org/tensorboard/r1/histograms
