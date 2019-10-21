---
title: tensorflow assign
date: 2019-05-08 20:48:06
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.assign
### 简单解释
op = x.assign(y)
将y的值赋值给x，执行sess.run(op)后，x的值就变成和y一样了。

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_assign.py)

``` python
import tensorflow as tf

# 声明两个Variable
x1 = tf.Variable([3,4])
x2 = tf.Variable([9,1])

# y是将x2 assign 给x1的op
y = x1.assign(x2)

with tf.Session() as sess:
  sess.run(tf.global_variables_initializer())
  xx1 = sess.run(x1)
  # 输出x1
  print(xx1)
  # [3 4]

  xx2 = sess.run(x2)
  # 输出x2
  print(xx2)
  # [9 1]

  print(sess.run(x1))
  # [3 4]
  
  # 执行y操作
  yy = sess.run(y)
  print(yy)
  # [9 1]
  
  # 发现x1已经用x2赋值了
  print(sess.run(x1))
  # [9 1]
  print(sess.run(x2))
  # [9 1]
```


