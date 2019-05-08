---
title: tensorflow summary
date: 2019-05-08 17:39:43
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.summary
### api
函数
tf.summary.scalar(name, tensor, collections=None, family=None)
定义一个summary标量

类
tf.summary.FileWriter(self, logdir,　graph=None, max_queue=10,flush_secs=120, graph_def=None, filename_suffix=None)
定义将数据写入文件的类

类内函数
tf.summary.FileWriter.add_summary(self, summary, global_step=None)
将summary类型变量转换为事件 


### 代码示例(无法运行)
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_summary.py)
``` python
import tensorflow as tf
summary_loss = tf.summary.scalar('loss', loss)
summary_weights = tf.summary.scalar('weights', weights)
writer = tf.summary.FileWriter("./summary/")
sess = tf.Session()
loss_, weights_ = sess.run([summary_loss, summary_weights], feed_dict={})
writer.add_summary(loss_)
writer.add_summary(weights_)
# 或者先把loss和weights merge 一下，然后再run
merged = tf.summary.merge_all()
merged_ = sess.rum([merged], feed_dict={})
writer.add_summary(merged_, global_step)
```

使用tensorboard --logdir ./summary/打开tensorboard


