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
config = ConfigProto()
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
在左上角选中
File\>\>Settings\>\>Build.Execution,Deployment\>\>Console\>\>Python Console
在Environment下的Environment variables中添加
LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
即可。



## 参考文献
1.https://github.com/tensorflow/tensorflow/issues/4842
2.https://github.com/tensorflow/tensorflow/issues/24496
3.https://github.com/tensorflow/tensorflow/issues/9530
