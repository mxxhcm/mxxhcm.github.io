---
title: tensorflow basic operation
date: 2019-05-08 18:57:41
tags:
 - tensorflow
 - python
categories: tensorflow
---

## 创建Session
``` python
import tensorflow as tf
import matplotlib.pyplot as plt
n = 32
x = tf.linspace(-3.0, 3.0, n)
```

### 普通Session
``` python
sess = tf.Session()
```


### 交互式Session 
``` python
import tensorflow as tf
sess = tf.InteractiveSession()
x.eval()
```

### 在sess内执行op
#### 方法1
sess.run(op)
``` python
result  = sess.run(x)
```
#### 方法2
op.eval()
``` python
x.eval(session=sess)
sess.close()
```

## 新op添加到默认图上
``` python
import tensorflow as tf
sigma = 1.0
mean = 0.0
# 和x的shape是一样的
z = (tf.exp(tf.negative(tf.pow(x - mean, 2.0) /
                        (2.0 * tf.pow(sigma, 2.0)))) *
     (1.0 / (sigma * tf.sqrt(2.0 * 3.1415))))
print(type(z))
print(z.graph is tf.get_default_graph())

plt.plot(z.eval())
plt.show()
```

## 查看shape
``` python
import tensorflow as tf
print(z.shape)
print(z.get_shape())
print(z.get_shape().as_list())
print(tf.shape(z).eval())
```

## 常用function
### tf.stack
``` python
import tensorflow as tf
print(tf.stack([tf.shape(z),tf.shape(z),[3]]).eval())
# tf.reshape, tf.matmul
z_ = tf.matmul(tf.reshape(z, (n, 1)), tf.reshape(z, (1, n)))
plt.imshow(z_.eval()) plt.show()
```

### tf.ones_like, tf.multiply
tf.ones_like返回与输入tensor具有相同shape的tensor
``` python
import tensorflow as tf
x = tf.reshape(tf.sin(tf.linspace(- 3.0, 3.0, n)), (n, 1))
print(x.shape)
y = tf.reshape(tf.ones_like(x), (1, n))
print(y.shape)
print(y.eval())
z = tf.multiply(tf.matmul(x,y), z_)
print(z.shape)
plt.imshow(z.eval())
plt.show()
```

## 列出graph中所有操作
``` python
import tensorflow as tf
ops = tf.get_default_graph().get_operations()
print([op for op in ops])
```
## 代码
[完整地址](https://github.com/mxxhcm/code/blob/master/tf/some_ops/tf_basic.py)

## 参考文献

