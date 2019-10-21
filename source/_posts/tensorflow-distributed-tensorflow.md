---
title: tensorflow distributed tensorflow
date: 2019-07-13 21:11:06
tags:
 - tensorflow
 - python
 - distributed tensorflow
categories: tensorflow
---

## Distributed Tensorflow
这篇文章主要介绍如何创建cluster of tensorflow serves，并且将一个computation graph分发到这个cluster上。

## Cluster和Server
### 简介
tensorflow中，一个cluster是一系列参与tensorflow graph distriuted execution的tasks。每个task和一个tensorflow server相关联，这个server可能包含一个"master"用来创建session，或者"worker"用来执行图中的op。一个cluster可以被分成更多jobs，每一个job包含一个或者更多个tasks。
为了创建一个cluster，需要给每一个task指定一个server。每一个task都运行在不同的devices上。在每一个task上，做以下事情：
1. 创建一个tf.train.ClusterSpec描述这个cluster的所有tasks，这对于所有的task都是一样的。
2. 创建一个tf.train.Server，需要的参数是tr.train.ClusterSpec，识别local task使用job name和task index。

### 创建一个tf.train.ClusterSpec
下面的表格中给出了两个cluster的示例。传入的参数是一个dictionary，key是job的name，value是该job的所有可用devices。第二列对应的task的scope name。
tf.train.ClusterSpec construction|Available tasks
--|--|--
tf.train.ClusterSpec({"local": ["localhost:2222", "localhost:2223"]}) | /job:local/task:0/job:local/task:1
tf.train.ClusterSpec({ "worker": [ "worker0.example.com:2222", "worker1.example.com:2222", "worker2.example.com:2222"], "ps": ["ps0.example.com:2222", "ps1.example.com:2222"]}) | /job:worker/task:0    /job:worker/task:1  /job:worker/task:2  /job:ps/task:0  /job:ps/task:1


### 为每一个task创建一个tf.train.Server instance
tf.train.Server对象包含local devices的集合，和其他tasks的connections

## 参考文献
1.https://github.com/tensorflow/examples/blob/master/community/en/docs/deploy/distributed.md
