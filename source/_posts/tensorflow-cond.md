---
title: tensorflow cond
date: 2019-05-10 17:01:14
tags:
 - tensorflow
 - python
categories: tensorflow
---

## tf.cond
### 一句话介绍
和if语句的功能和很像，如果条件为真，返回一个函数，如果条件为假，返回另一个函数。

### API
``` python
tf.cond(
    pred, # 条件
    true_fn=None, # 如果条件为真，执行该函数
    false_fn=None, # 如果条件为假，执行该函数
    strict=False,
    name=None,
    fn1=None,
    fn2=None
)
```
最后返回的是true_fn或者false_fn返回的还是tf.Tensor类型的变量。

### 代码示例1
[代码地址]()
``` python
import tensorflow as tf
import numpy as np


x = tf.placeholder(tf.int32, [10])
y = tf.constant([10, 3.2])

# for i in range(10):
#     if tf.equal(x[i], 0):
#         y = tf.add(y, 1)
#     else:
#         y = tf.add(y, 10)

# 上面的代码起到了和下面代码相同的作用，但是上面的代码在tensorflow中会报错，不能运行，因为x[i]==0返回的不是python的bool类型，而是bool类型的tf.Tensor。
# TypeError: Using a tf.Tensor as a Python bool is not allowed.

for i in range(10):
    y = tf.cond(tf.equal(x[i], 0), lambda: tf.add(y, 1), lambda: tf.add(y, 10))

result = tf.log(y)

with tf.Session() as sess:
   inputs = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
   print(sess.run(result, feed_dict={x: inputs}))

```

### 代码示例2
``` python
def myfunc(x):
   if (x > 0):
      return 1
   return 0


with tf.Session() as sess:
    x = tf.constant(4)
    # print(myfunc(x))
    # raise TypeError("Using a `tf.Tensor` as a Python `bool` is not allowed. "
    # TypeError: Using a `tf.Tensor` as a Python `bool` is not allowed. Use `if t is not None:` instead of `if t:` to test if a tensor is defined, and use TensorFlow ops such as tf.cond to execute subgraphs conditioned on the value of a tensor.
    result = tf.cond(tf.greater(x, 0), lambda: 1, lambda: 0)
    print(type(result))
    print(result.eval())
```
上述代码中定义了一个函数，实现判断某个值是否大于0。但是这个函数是错误的，因为$x\gt 0$返回一个bool类型的tf.Tensor不能用作if的判断条件，所以需要使用tf.cond语句。

### 代码示例3
``` python
# Example 3
x = tf.constant(4)
y = tf.constant(4)

with tf.Session() as sess:
    print(x) 
    print(y) 
    if x == y:
      print(True)
    else:
      print(False)
    result = tf.equal(x, y)
    print(result.eval())
    def f1(): 
      print("f1 declare")
      return [1, 1]
    def f2():
      print("f2 declare")
      return [0, 0]
    res = tf.cond(tf.equal(x, y), f1, f2)
    print(res)
```

## 参考文献
1.https://www.tensorflow.org/api_docs/python/tf/cond
2.https://stackoverflow.com/questions/48571521/tensorflow-error-using-a-tf-tensor-as-a-python-bool-is-not-allowed
3.https://blog.csdn.net/Cerisier/article/details/79819248 
