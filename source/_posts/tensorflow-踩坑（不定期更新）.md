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

## 参考文献
1.https://github.com/tensorflow/tensorflow/issues/4842
