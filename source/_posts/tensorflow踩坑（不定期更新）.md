---
title: tensorflow踩坑（不定期更新）
date: 2019-03-07 14:51:01
tags:
 - 机器学习
 - 深度学习
 - tensorflow
categories: python
---


## 问题
1. TypeError: The value of a feed cannot be a tf.Tensor object
Sess.run(train, feed_dict={x:images, y:labels}的输入不能是tensor，可以使用sess.run(tensor)得到numpy.array形式的数据再喂给feed_dict。
> Once you have launched a sess, you can use your_tensor.eval(session=sess) or sess.run(your_tensor) to get you feed tensor into the format of numpy.array and then feed it to your placeholder.

## 用法
### tf.app.flags
``` python
flags.py
import tensorflow as tf

flags = tf.app.flags
flags.DEFINE_string('model', 'mxx', 'Type of model')
flags.DEFINE_boolean('gpu','True', 'use gpu?')
FLAGS = flags.FLAGS

def main(_):
    for k,v in FLAGS.flag_values_dict().items():
        print(k, v)

if __name__ == "__main__":
    tf.app.run(main)
```
传递参数的方法有两种，一种是命令行~$:python flags.py --model hhhh ，一种是pycharm中传递参数。

### tf.train.Saver()
#### 代码示例(可运行)
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

### tf.summary
#### api
函数
tf.summary.scalar(name, tensor, collections=None, family=None)
定义一个summary标量

类
tf.summary.FileWriter(self, logdir,　graph=None, max_queue=10,flush_secs=120, graph_def=None, filename_suffix=None)
定义将数据写入文件的类

类内函数
tf.summary.FileWriter.add_summary(self, summary, global_step=None)
将summary类型变量转换为事件 


#### 代码示例(无法运行)
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

### tf.multinomial
多项分布，采样。
如下[示例](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_multinominal.py)
``` python
import tensorflow as tf

# tf.multinomial(logits, num_samples, seed=None, name=None)
# logits 是一个二维张量，指定概率，num_samples是采样个数
sess = tf.Session()
sample = tf.multinomial([[5.0, 5.0, 5.0], [5.0, 4, 3]], 10) # 注意logits必须是float
for _ in range(5):
  print(sess.run(sample))
```
输出结果如下:
> [[2 1 2 1 0 2 1 1 1 0]
 [1 0 0 1 0 1 0 1 0 0]]
[[2 2 0 2 2 0 2 0 1 2]
 [1 0 0 2 0 1 0 1 1 0]]
[[0 0 0 2 0 0 1 2 0 1]
 [0 0 0 1 0 1 0 0 0 0]]
[[2 1 0 1 1 1 0 0 2 0]
 [1 0 0 2 0 0 0 0 0 1]]
[[1 0 1 0 0 1 2 2 0 0]
 [1 0 0 0 0 1 1 1 2 0]]

### tf.max
#### tf.maximum
比较两个tensor，返回element-wise两个tensor的最大值。
如下[示例](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_maximum.py)：
``` python
import tensorflow as tf

sess = tf.Session()
a = tf.Variable([1, 2, 3])
b = tf.Variable([2, 1, 4])

sess.run(tf.global_variables_initializer())
print("a: ", sess.run(a))
print("b: ", sess.run(b))
c = tf.maximum(a, b)

print("tf.maximum(a, b):\n  ", sess.run(c))
```
输出如下：
> a:  [1 2 3]
b:  [2 1 4]
tf.maximum(a, b):
   [2 2 4]

## 参考文献
1.https://github.com/tensorflow/tensorflow/issues/4842
2.https://www.bilibili.com/read/cv681031/
3.https://cv-tricks.com/tensorflow-tutorial/save-restore-tensorflow-models-quick-complete-tutorial/
