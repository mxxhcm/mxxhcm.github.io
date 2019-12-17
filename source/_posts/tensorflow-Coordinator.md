---
title: tensosrflow Coordinator
date: 2019-07-13 20:17:13
tags:
 - tensorflow
 - python
categories: tensorflow
---


## Coordinator
### 简介
这个类用来协调多个thread同时工作，同时停止。常用的方法有：
- should_stop()：如果满足thread停止条件的话，返回True
- request_stop()：请求thread停止，调用该方法后，should_stop()返回True
- join(<list of threads>)：等待所有的threads停止。

### 理解
其实我觉得这个类的作用好像没有那么大，或者我没有用到这种场景。反正就是满足thread的终止条件时，调用request_stop()函数，让should_stop()返回True。
## 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tf/ops/tf_train_Coordinator.py)
``` python
import tensorflow as tf
import time
import random
import threading

n = 0

def add(index):
    global n
    while not coord.should_stop():
    #while True:
        if n > 10:
            print(index, " done")
            coord.request_stop()
            #break
        print("before A, thread ", index, n)
        n += 1
        print("after A, thread ", index, n)
        time.sleep(random.random())
        print("before B, thread ", index, n)
        n += 1
        print("after B, thread ", index, n)
    
if __name__ == "__main__":
    with tf.Session() as sess:
        coord = tf.train.Coordinator()

        jobs = []
        for i in range(2):
            jobs.append(threading.Thread(target=add, args=(i,)))

        for j in jobs:
            j.start()

        coord.join(jobs)
        print("Hello World!")
```

## 参考文献
1.https://wiki.jikexueyuan.com/project/tensorflow-zh/how_tos/threading_and_queues.html
