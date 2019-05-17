---
title: tensorflow saver
date: 2019-05-08 17:39:54
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.train.Saver()
Saver是类，不是函数。可以用来保存和恢复variable，保存和恢复模型。Saver对象提供save和restore等各类op。

## API
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

## 保存和全部variables
- 恢复variable时，无需初始化。
- 恢复variable时，使用的是variable的name，不是op的name。只要知道variable的name即可。save和restore的op name不需要相同，只要variable name相同即可。
- 对于使用tf.Variable()创建的variable，如果没有指定variable名字的话，系统会为其生成默认名字，在恢复的时候，需要使用tf.get_variable()恢复variable，同时传variable name和shape。

### 保存全部variable
``` python
saver = tf.train.Saver()
saver.save(sess, save_path) # 需要指定的是checkpoint的名字而不是目录
```
### 恢复全部variable
``` python
saver = tf.train.Saver()
saver.restore(sess, save_path)
```

## 保存和部分variables
### 保存全部variable
``` python
saver = tf.train.Saver({"variable_name1": op_name1,..., "variable_namen": op_namen})
saver.save(sess, save_path) # 需要指定的是checkpoint的名字而不是目录
```

### 恢复全部variable
``` python
saver = tf.train.Saver({"variable_name1": op_name1,..., "variable_namen": op_namen})
saver.restore(sess, save_path)
```

## 查看checkpoint文件中的variable
使用inspect_ckeckpoint库
``` python
# import the inspect_checkpoint library
from tensorflow.python.tools import inspect_checkpoint as chkp

# 打印checkpoint文件中所有variable
chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='', all_tensors=True)

chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='v1', all_tensors=False)

chkp.print_tensors_in_checkpoint_file("saver/variables/all_variables.ckpt", tensor_name='v2', all_tensors=False)
```

## 保存和恢复模型
其实和保存，恢复模型没有什么区别。只是把整个模型的variables都save和restore了。

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


## 参考文献
1.https://www.tensorflow.org/guide/saved_model
2.https://www.tensorflow.org/api_docs/python/tf/train/Saver
3.https://www.bilibili.com/read/cv681031/
4.https://cv-tricks.com/tensorflow-tutorial/save-restore-tensorflow-models-quick-complete-tutorial/
