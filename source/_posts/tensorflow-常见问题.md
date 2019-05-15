---
title: tensorflow 常见问题（不定期更新）
date: 2019-03-07 14:51:01
tags:
 - 机器学习
 - 深度学习
 - tensorflow
 - 常见问题
 - python
categories: tensorflow
---


## 问题1

### 报错
``` txt
TypeError: The value of a feed cannot be a tf.Tensor object
```

### 问题原因
sess.run(op, feed_dict={})中的feed value不能是tf.Tensor类型。

### 解决方法
sess.run(train, feed_dict={x:images, y:labels}的输入不能是tensor，可以使用sess.run(tensor)得到numpy.array形式的数据再喂给feed_dict。
> Once you have launched a sess, you can use your_tensor.eval(session=sess) or sess.run(your_tensor) to get you feed tensor into the format of numpy.array and then feed it to your placeholder.

## 问题2
### 配置
环境配置如下：
- Ubuntu 18.04
- CUDA 10.0
- CuDNN 7.4.2
- Python3.7.3
- Tensorflow 1.13.1
- Nvidia Drivers 430.09
- RTX2070

### 报错
``` txt
...
2019-05-12 14:45:59.355405: E tensorflow/stream_executor/cuda/cuda_dnn.cc:334] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2019-05-12 14:45:59.357698: E tensorflow/stream_executor/cuda/cuda_dnn.cc:334] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
...
```

### 问题原因
GPU不够用了。

### 解决方法
在代码中添加下面几句：
``` python
config = tf.ConfigProto()
config.gpu_options.allow_growth = True
session = InteractiveSession(config=config)
```

## 问题3
在命令行或者pycharm中import tensorflow报错
### 报错
``` txt
ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory
Failed to load the native TensorFlow runtime.
```

### 问题原因
没有配置CUDA环境变量

### 解决方法
#### 命令行中
在.bashrc文件中加入下列语句：
``` shell
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```

#### pycharm中
##### 方法1（这种方法我没有实验成功，不知道为什么）
在左上角选中
File\>\>Settings\>\>Build.Execution,Deployment\>\>Console\>\>Python Console
在Environment下的Environment variables中添加
LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}即可。
##### 方法2
修改完.bashrc文件后从终端中运行pycharm。

## 问题4
sess.run(op, feed_dict={})中feed的数据中包含有list的时候会报错。

### 报错
``` txt
TypeError: unhashable type: 'list'
```
### 问题原因
feed_dict中不能的value不能是list。

### 解决方法
``` python
feed_dict = {
               placeholder : value 
                  for placeholder, value in zip(placeholder_list, inputs_list))
            }

```

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_placeholder_list.py)

## 问题5
执行mnist_with_summary代码时报错
### 报错
``` txt
I tensorflow/stream_executor/dso_loader.cc:142] Couldn't open CUDA library libcupti.so.10.0. LD_LIBRARY_PATH: /usr/local/cuda/lib64:
2019-05-13 23:04:10.620149: F tensorflow/stream_executor/lib/statusor.cc:34] Attempting to fetch value instead of handling error Failed precondition: could not dlopen DSO: libcupti.so.10.0; dlerror: libcupti.so.10.0: cannot open shared object file: No such file or directory
Aborted (core dumped)
```

### 原因
libcupti.so.10.0包没找到

### 解决方法
执行以下命令，找到相关的依赖包：
~$:find /usr/local/cuda/ -name libcupti.so.10.0 
输出如下：
> /usr/local/cuda/extras/CUPTI/lib64/libcupti.so.10.0

然后修改~/.bashrc文件中相应的环境变量:
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64/:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} 
重新运行即可。


## 问题6
tf.Session()和tf.InteractiveSession()混用问题。
### 报错
``` txt
tensorflow.python.framework.errors_impl.FailedPreconditionError: Attempting to use uninitialized value prediction/l1/w
	 [[{{node prediction/l1/w/read}}]]
	 [[{{node prediction/LogSoftmax}}]]
```

### 问题原因
声明了如下session:
``` python
sess = tf.Session()
sess.run(tf.global_variables_initializer())
```
在接下来的代码中，因为我声明的是tf.Session()，使用了op.eval()函数，这种用法是tf.InteractiveSession的用法，所以就相当于没有初始化。
result = op.eval(feed_dict={})
然后就报了未初始化的错误。
把代码改成：
result = sess.run([op], feeed_dct={})
即可，即上下文使用的session应该一致。

### 解决方案
使用统一的session类型

## 问题7
feed_dict必须是numpy.ndarray，不能是其他类型，尤其不能是tf.Variable。

### 报错
``` txt
valueerror setting an array element with a sequence,
```

### 解决方法
检查sess.run(op, feed_dict={})中的feed_dict，确保他们的类型，不能是tf.Variable()类型的对象，需要是numpy.ndarray。

## 问题8
如何获得tf.Variable()对象的值

### 解决方法
``` python
import tensorflow as tf
x = tf.Varialbe([1.0, 2.0])
sess = tf.Session()
sess.run(tf.global_variables_initializer())
value = sess.run(x)
```
或者
```
import tensorflow as tf
x = tf.Varialbe([1.0, 2.0])
sess = tf.InteractiveSession()
sess.run(tf.global_variables_initializer())
x.eval()
```


## 参考文献
1.https://github.com/tensorflow/tensorflow/issues/4842
2.https://github.com/tensorflow/tensorflow/issues/24496
3.https://github.com/tensorflow/tensorflow/issues/9530
4.https://stackoverflow.com/questions/51128427/how-to-feed-list-of-values-to-a-placeholder-list-in-tensorflow
5.https://github.com/tensorflow/tensorflow/issues/11897
6.https://stackoverflow.com/questions/34156639/tensorflow-python-valueerror-setting-an-array-element-with-a-sequence-in-t
7.https://stackoverflow.com/questions/33679382/how-do-i-get-the-current-value-of-a-variable
