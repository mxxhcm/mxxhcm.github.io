---
title: tensorflow 常见问题（不定期更新）
date: 2019-03-07 14:51:01
tags:
 - 机器学习
 - tensorflow
 - 常见问题
 - python
categories: tensorflow
---


## 问题1-The value of a feed cannot be a tf.Tensor object
### 报错
``` txt
TypeError: The value of a feed cannot be a tf.Tensor object
```

### 问题原因
sess.run(op, feed_dict={})中的feed value不能是tf.Tensor类型。

### 解决方法
sess.run(train, feed_dict={x:images, y:labels}的输入不能是tensor，可以使用sess.run(tensor)得到numpy.array形式的数据再喂给feed_dict。
> Once you have launched a sess, you can use your_tensor.eval(session=sess) or sess.run(your_tensor) to get you feed tensor into the format of numpy.array and then feed it to your placeholder.

## 问题2-Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR

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

## 问题3-libcublas.so.10.0: cannot open shared object file: No such file or directory

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

## 问题4-dlerror: libcupti.so.10.0: cannot open shared object file: No such file or directory

执行mnist_with_summary代码时报错
### 报错
``` txt
I tensorflow/stream_executor/dso_loader.cc:142] Couldn't open CUDA library libcupti.so.10.0. LD_LIBRARY_PATH: /usr/local/cuda/lib64:
2019-05-13 23:04:10.620149: F tensorflow/stream_executor/lib/statusor.cc:34] Attempting to fetch value instead of handling error Failed precondition: could not dlopen DSO: libcupti.so.10.0; dlerror: libcupti.so.10.0: cannot open shared object file: No such file or directory
Aborted (core dumped)
```

### 问题问题问题问题问题问题问题问题问题原因
libcupti.so.10.0包没找到

### 解决方法
执行以下命令，找到相关的依赖包：
~$:find /usr/local/cuda/ -name libcupti.so.10.0 
输出如下：
> /usr/local/cuda/extras/CUPTI/lib64/libcupti.so.10.0

然后修改~/.bashrc文件中相应的环境变量:
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64/:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} 
重新运行即可。


## 问题5-unhashable type: 'list'

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


## 问题6-Attempting to use uninitialized value 

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

## 问题7-setting an array element with a sequence
feed_dict键值对中中值必须是numpy.ndarray，不能是其他类型。

### 报错
``` txt
value error setting an array element with a sequence,
```
### 问题原因
feed_dict中key-value的value必须是numpy.ndarray，不能是其他类型，尤其不能是tf.Variable。

### 解决方法
检查sess.run(op, feed_dict={})中的feed_dict，确保他们的类型，不能是tf.Variable()类型的对象，需要是numpy.ndarray。

## 问题8-访问tf.Variable()的值
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

## 问题9-Can not convert a ndarray into a Tensor or Operation

### 报错
``` txt
Can not convert a ndarray into a Tensor or Operation.
```

### 问题原因
原因是sess.run()前后参数名重了，比如outputs = sess.run(outputs)，outputs本来是自己定义的一个op，但是sess.run(outputs)之后outputs就成了一个变量，就把定义的outputs op覆盖了。

### 解决方法
换个变量名字就行

## 问题10-本地使用gpu server的tensorboard
### 问题描述
在gpu server跑的实验结果，然后summary的记录也在server上，但是又没办法可视化，只好在本地可视化。

### 解决方法
使用ssh进行映射好了。
#### 本机设置
~$:ssh -L 12345:10.1.114.50:6006 mxxmhh@127.0.0.1
将本机的12345端口映射到10.1.114.50的6006端口，中间服务器使用的是本机。
或者可以使用10.1.114.50作为中间服务器。
~$:ssh -L 12345:10.1.114.50:6006 liuchi@10.1.114.50
或者可以使用如下方法：
~$:ssh -L 12345:127.0.0.1:6006 liuchi@10.1.114.50
从这个方法中，可以看出127.0.0.1这个ip是中间服务器可以访问的ip。
以上三种方法中，-L后的端口号12345可以随意设置，只要不冲突即可。

#### 服务端设置
然后在服务端运行以下命令：
~$:tensorboard --logdir logdir -port 6006
这个端口号也是可以任意设置的，不冲突即可。

#### 运行
然后在本机访问
[https://127.0.0.1:12345](https://127.0.0.1:12345)即可。


## 问题11-每一步summary一个list的每一个元素

### 问题原因
有一个tf list的placeholder，但是每一步只能生成其中的一个元素，所以怎么样summary中其中的某一个？

### 解决方法
``` shell
import tensorflow as tf
import numpy as np

number = 3
x_ph_list = []
for i in range(number):
    x_ph_list.append(tf.placeholder(tf.float32, shape=None))

x_summary_list = []
for i in range(number):
    x_summary_list.append(tf.summary.scalar("x%s" % i, x_ph_list[i]))

writer = tf.summary.FileWriter("./tf_summary/scalar_list_summary/sep")
with tf.Session() as sess:
    scope = 10
    inputs = np.arange(scope*number)
    inputs = inputs.reshape(scope, number)
    # inputs = np.random.randn(scope, number)
    for i in range(scope):
        for j in range(number):
            out, xj_s = sess.run([x_ph_list[j], x_summary_list[j]], feed_dict={x_ph_list[j]: inputs[i][j]})
            writer.add_summary(xj_s, global_step=i)
```

## 问题12- for value in summary.value: AttributeError: 'list' object has no attribute 'value'

### 问题描述
writer.add_summary时报错

### 报错
``` txt
File "/home/mxxmhh/anaconda3/lib/python3.7/site-packages/tensorflow/python/summary/writer/writer.py", line 127, in add_summary
    for value in summary.value:
AttributeError: 'list' object has no attribute 'value'
```

### 问题原因
执行以下代码
``` python
    s_ = sess.run([loss_summary], feed_dict={p_losses_ph: inputs1, q_losses_ph: inputs2})
    writer.add_summary(s_, global_step=1)
```
因为[loss_summary]加了方括号，就把它当成了一个list。。返回值也是list，就报错了

### 解决方法
- 方法1，在等号左边加一个逗号，取出list中的值
``` python
    s_, = sess.run([loss_summary], feed_dict={p_losses_ph: inputs1, q_losses_ph: inputs2})
```
- 方法2，去掉loss_summary外面的中括号。
``` python
    s_ = sess.run(loss_summary, feed_dict={p_losses_ph: inputs1, q_losses_ph: inputs2})
```

## 问题13- tf.get_default_session() always returns None type:

### 问题描述
调用tf.get_default_session()时，返回的是None

### 报错
``` txt
    tf.get_default_session().run(y)
AttributeError: 'NoneType' object has no attribute 'run'
```

### 问题原因
只有在设定default session之后，才能使用tf.get_default_session()获得当前的默认session，在我们写代码的时候，一般会按照下面的方式写：
``` python
import tensorflow as tf

with tf.Session() as sess:
    some operations
    ... 
```
这种情况下已经把tf.Session()生成的session当做了默认session，但是如果仅仅使用以下代码：
``` python
import tensorflow as tf

sess =  tf.Session()
sess.run(some operations)
... 
```
是没有把tf.Session()当成默认session的，即只有在with block内，才会将这个session当做默认session。

### 解决方案


 

## 参考文献
1.https://github.com/tensorflow/tensorflow/issues/4842
2.https://github.com/tensorflow/tensorflow/issues/24496
3.https://github.com/tensorflow/tensorflow/issues/9530
4.https://stackoverflow.com/questions/51128427/how-to-feed-list-of-values-to-a-placeholder-list-in-tensorflow
5.https://github.com/tensorflow/tensorflow/issues/11897
6.https://stackoverflow.com/questions/34156639/tensorflow-python-valueerror-setting-an-array-element-with-a-sequence-in-t
7.https://stackoverflow.com/questions/33679382/how-do-i-get-the-current-value-of-a-variable
8.https://blog.csdn.net/michael__corleone/article/details/79007425
9.https://stackoverflow.com/questions/47721792/tensorflow-tf-get-default-session-after-sess-tf-session-is-none
