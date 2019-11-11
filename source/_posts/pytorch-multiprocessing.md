---
title: pytorch multiprocessing
date: 2019-05-08 21:52:42
tags:
 - pytorch
 - python
 - multiprocessing
 - 进程
 - 线程
categories: pytorch
mathjax: true
---

## torch.multiprocessing
### join
等待调用join()方法的线程执行完毕，然后继续执行。
可参见github[官方demo](https://github.com/mxxhcm/myown_code/tree/master/pytorch/tutorials/multiprocess_torch/mnist_hogwild)。

### share_memory\_()
在多个线程之间共享参数，如下[代码](https://github.com/mxxhcm/code/blob/master/pytorch/tutorials/torch_multiprocess_torch/share_memory.py)所示。可以用来实现A3C。
``` python
import torch.multiprocessing as mp
import torch
import time
import os


def proc(sec, x):
   print(os.getpid(),"  ", x)
   time.sleep(sec)
   print(os.getpid(), "  ", x)
   x += sec
   print(str(os.getpid()) + "  over.  ", x)


if __name__ == '__main__':
   num_processes = 3
   processes = []
   x = torch.ones([3,])
   x.share_memory_()
   for rank in range(num_processes):
     p = mp.Process(target=proc, args=(rank + 1, x))
     p.start() 
     processes.append(p)
   for p in processes:
     p.join()
   print(x)
```
输出结果如下所示：
> python share_memory.py 
7739    tensor([1., 1., 1.])
7738    tensor([1., 1., 1.])
7737    tensor([1., 1., 1.])
7737    tensor([1., 1., 1.])
7737  over.   tensor([2., 2., 2.])
7738    tensor([2., 2., 2.])
7738  over.   tensor([4., 4., 4.])
7739    tensor([4., 4., 4.])
7739  over.   tensor([7., 7., 7.])
tensor([7., 7., 7.])

我们可以发现$7739$这个线程中，传入的$x$还是和最开始的一样，但是在$7738$线程更新完$x$之后，$7739$使用的$x$就已经变成了更新后的$x$。所以，我猜测这里面应该是有一个对$x$的锁，保证$x$在同一时刻只能被一个线程访问。

## 参考文献
1.https://pytorch.org/docs/stable/multiprocessing.html

