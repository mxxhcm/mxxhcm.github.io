---
title: tensorflow saver代码及例子
date: 2019-05-08 17:39:54
tags:
- tensorflow
- python
categories: python
---

## tf.train.Saver()
### 代码示例(可运行)
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
1.https://www.bilibili.com/read/cv681031/
2.https://cv-tricks.com/tensorflow-tutorial/save-restore-tensorflow-models-quick-complete-tutorial/
