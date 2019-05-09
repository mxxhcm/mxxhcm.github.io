---
title: tensorflow math
date: 2019-05-08 17:38:46
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.math
- tf.add(x, y, name=None) # 求和
- tf.sub(x, y, name=None) # 减法
- tf.mul(x, y, name=None) # 乘法
- tf.div(x, y, name=None) # 除法
- tf.mod(x, y, name=None) # 取模
- tf.maximumd(x, y, name=None) # x \> y?x:y
- tf.minimum(x, y, name=None) # x \< y?x:y
- tf.abs(x, name=None) # 求绝对值
- tf.neg(x, name=None) # 取负
- tf.sign(x, name=None) # 返回符号
- tf.inv(x, name=None) # 取反
- tf.square(x, name=None) # 平方
- tf.round(x, name=None) # 四舍五入
- tf.sqrt(x, name=None) # 开根号
- tf.pow(x, name=None) # 
- tf.exp(x, name=None) # 
- tf.log(x, name=None) # 
- tf.sin(x, name=None) # 
- tf.cos(x, name=None) # 
- tf.tan(x, name=None) # 
- tf.atan(x, name=None) # 

## 代码示例
### tf.maximum
比较两个tensor，返回element-wise两个tensor的最大值。
[代码地址示例](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_maximum.py)：
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


