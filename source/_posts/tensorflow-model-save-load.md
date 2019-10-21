---
title: tensorflow model save load
date: 2019-05-09 15:14:26
tags:
 - tensorflow
 - python
 - Saver
 - save
 - restore
categories: tensorflow
---

## tf.train.Saver保存和恢复模型
``` python
saver = tf.train.Saver()
saver.save()
```
调用上述代码之后会存存储以下几个文件：
``` txt
checkpoint
model.ckpt.data-00000-of-00001
model.ckpt.index
model.ckpt.meta
```
其中checkpoint文件存储的是最近保存的文件的名字，meta文件存放的是计算图的定义，index和data文件存放的是权重文件。


下面介绍一下上述代码中出现的两个API，tf.train.Saver()和tf.train.Saver().save()。
### tf.train.Saver()
Saver是类，不是函数。可以用来保存，恢复variable和model，Saver对象提供save()和restore()等函数，save()保存模型，restore()加载模型。
``` python
__init__(
    var_list=None, # 指定要保存的variablelist
    reshape=False,
    sharded=False,
    max_to_keep=5, # 最多保留最近的几个checkpoints
    keep_checkpoint_every_n_hours=10000.0,
    name=None,
    restore_sequentially=False,
    saver_def=None,
    builder=None,
    defer_build=False,
    allow_empty=False,
    write_version=tf.train.SaverDef.V2,
    pad_step_number=False,
    save_relative_paths=False,
    filename=None
)
```

### tf.train.Saver.save()
``` python
save(
    sess,       \\传入当前要保存的session
    save_path,  \\指定checkpoint的路径
    global_step=None,   \\当前存的model的step
    latest_filename=None,
    meta_graph_suffix='meta',
    write_meta_graph=True,  \\指定是否要保存计算图
    write_state=True,
    strip_default_attrs=False,
    save_debug_info=False
)
```
这里说一下save_path，如果不指定的话，文件名默认是空的，在linux下是以.开头的（即当前目录），所以会显示成隐藏文件。通常情况下我们指定checkpoint要保存的路径，以及名字，比如叫model.ckpt，在load的时候还使用这个名字就行。指定了global_step之后，tf会自动在路径后面加上step进行区分。


## 读取graph
### 读取图的定义
meta文件中存放了计算图的定义，可以直接使用API tf.train.import_meta_graph()函数调用：
``` python
import tensorflow as tf
with tf.Session() as sess:
    saver = tf.train.import_meta_graph("model.ckpt.meta")
```
这时计算图就已经定义在当前sess中了。上述代码会保留原始的device信息，如果迁移到其他设备时，可能由于没有指定设备出错，这个问题可以通过指定一个特殊的参数clear_devices解决：
``` python
import tensorflow as tf
with tf.Session() as sess:
    saver = tf.train.import_meta_graph("model.ckpt.meta", clear_devices=True)
```
这样子就和device无关了。

### 访问graph中的参数
#### 通过collection访问计算图中collection的键
这里的键指的是graph中都有哪些[collections]()。
- ``` python
print(sess.graph.get_all_collection_keys())
```
- ``` python
print(sess.graph.collections)
```
- ``` python
tf.get_default_graph().get_all_collection_keys()
```

#### 访问collection
- ``` python
sess.graph.get_collection("summaries")
```
- ``` python
tf.get_collection("")
```

示例
``` python
import tensorflow as tf

#saver = tf.train.Saver()
with tf.Session() as sess:
    new_saver = tf.train.import_meta_graph("saver1.ckpt.meta")
    print(sess.graph)
    for var in tf.get_collection(tf.GraphKeys.TRAINABLE_VARIABLES):
        print(var)
```

#### 通过operation访问
- ``` python
sess.graph.get_opeartions()
```
- ``` python
for op in sess.graph.get_opeartions():
    print(op.name, op.values())
```
- ``` python
sess.graph.get_operation_by_name("op_name").node_def
``` 

## 保存和恢复variables
### 保存和恢复全部variables
- 恢复variable时，无需初始化。
- 恢复variable时，使用的是variable的name，不是op的name。只要知道variable的name即可。save和restore的op name不需要相同，只要variable name相同即可。
- 对于使用tf.Variable()创建的variable，如果没有指定variable名字的话，系统会为其生成默认名字，在恢复的时候，需要使用tf.get_variable()恢复variable，同时传variable name和shape。

#### 保存全部variables
``` python
saver = tf.train.Saver()
saver.save(sess, save_path) # 需要指定的是checkpoint的名字而不是目录
```

#### 恢复全部variables
``` python
saver = tf.train.Saver()
saver.restore(sess, save_path)
```

### 保存和恢复部分variables
#### 保存全部variable
``` python
saver = tf.train.Saver({"variable_name1": op_name1,..., "variable_namen": op_namen})
saver.save(sess, save_path) # 需要指定的是checkpoint的名字而不是目录
```

#### 恢复全部variable
``` python
saver = tf.train.Saver({"variable_name1": op_name1,..., "variable_namen": op_namen})
saver.restore(sess, save_path)
```

## 保存和恢复模型
其实和保存恢复变量没有什么区别。只是把整个模型的variables都save和restore了。

## 代码示例
[代码地址](https://github.com/mxxhcm/code/tree/master/tf/some_ops/saver_restore)
``` python
import tensorflow as tf

graph = tf.Graph()
with graph.as_default():
    W = tf.Variable([0.3], dtype=tf.float32)
    b = tf.Variable([-0.3], dtype=tf.float32)

    # input and output
    x = tf.placeholder(tf.float32)
    y = tf.placeholder(tf.float32)
    predicted_y = W*x+b

    # MSE loss
    loss = tf.reduce_mean(tf.square(y - predicted_y))
    # optimizer
    optimizer = tf.train.GradientDescentOptimizer(0.01)
    train_op = optimizer.minimize(loss)

inputs = [1, 2, 3, 4]
outputs = [2, 3, 4, 5]

with tf.Session(graph=graph) as sess:
    saver = tf.train.Saver()
    init_op = tf.global_variables_initializer()
    sess.run(init_op)
    for i in range(5000):
        sess.run(train_op, feed_dict={x: inputs, y: outputs})
    l_, W_, b_ = sess.run([loss, W, b], feed_dict={x: inputs, y: outputs})
    print("loss: ", l_, "w: ", W_, "b:", b_)
    checkpoint = "./checkpoint/saver1.ckpt"
    save_path = saver.save(sess, checkpoint)
    print("Model has been saved in %s." % save_path)

with tf.Session(graph=graph) as sess:
    saver = tf.train.Saver()
    saver.restore(sess, checkpoint)
    l_, W_, b_ = sess.run([loss, W, b], feed_dict={x: inputs, y: outputs})
    print("loss: ", l_, "w: ", W_, "b:", b_)
    print("Model has been restored.")
```

## 获取最新的checkpoint文件
### tf.train.get_checkpoint_state()
给出checkpoint文件所在目录，可以使用get_checkpoint_state()获得最新的checkpoint文件：
``` python
ckpt = tf.train.get_checkpoint_state(checkpoint_dir)
if ckpt and ckpt.model_checkpoint_path:
    save.restore(sess, ckpt.model_checkpoint_path)
```

### 使用inspect_checkpoint库
``` python
# import the inspect_checkpoint library
from tensorflow.python.tools import inspect_checkpoint as chkp

# 打印checkpoint文件中所有variable
chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='', all_tensors=True)

# 打印变量"v1"
chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='v1', all_tensors=False)

chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='v2', all_tensors=False)
```

## 模型的冻结
模型的冻结是不在训练模型，只用于正向推导，所以把变量转换成常量后，和计算图一起保存在协议缓冲区文件(.pb)文件中，因此需要在计算图中预先定义输出节点的名称，示例如下：
``` python
import tensorflow as tf

output_nodes = ["Accuracy/prediction", "Metric/Dice"]

# 加载计算图
saver = tf.train.import_meta_graph("model.ckpt.meta", clear_devices=True)

with tf.Session() as sess:
    input_graph_def = sess.graph.as_graph_def()
    # load model
    saver.restore(sess, "model.ckpt")
    # 将变量转换为常量
    output_graph_def = tf.graph_util.convert_variables_to_constants(sess, input_graph_def, output_nodes)
    # 写入pb文件
    with open("frozen_model.pb", "wb") as f:
        f.write(output_graph_def.SerializeToString())
```

## 模型的执行
从协议缓冲区文件(.pb)文件中读取模型，导入计算图
``` python
# 读取模型并保存到序列化模型对象中
with open(frozen_graph_path, "rb") as f:
    graph_def = tf.GraphDef()
    graph_def.ParseFromString(f.read())
# 导入计算图
graph = tf.Graph()
with graph.as_default():
    tf.import_graph_def(graph_def, name="Test")
```

获取输入和输出的张量，然后将测试数据feed给输入张量，得到结果。
``` python
x_tensor = graph.get_tensor_by_name("Test/input/image-input:0")
y_tensor = graph.get_tensor_by_name("Test/input/label-input:0")
keep_prob = graph.get_tensor_by_name("Test/dropout/Placeholder:0")
acc_op = graph.get_tensor_by_name("Test/accuracy/prediction:0")

from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("mnist_data", one_hot=True)
x_values, y_values = mnist.test.next_batch(10000)

with tf.Session(graph=graph) as sess:
    accuracy = sess.run(acc_op, feed_dict={x_tensor: x_values,
                                          y_tensor: y_values,
                                          keep_prob: 1.0})
    print(accuracy)
```

## 参考文献
1.https://www.jarvis73.cn/2018/04/25/Tensorflow-Model-Save-Read/
2.https://www.tensorflow.org/guide/saved_model
3.https://www.tensorflow.org/api_docs/python/tf/train/Saver
4.https://www.bilibili.com/read/cv681031/
5.https://cv-tricks.com/tensorflow-tutorial/save-restore-tensorflow-models-quick-complete-tutorial/
