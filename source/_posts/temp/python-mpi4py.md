---
title: python mpi4py
date: 2019-10-08 17:25:46
tags:
 - python
 - mpi4py
 - 多进程
categories: python
---

## MPI
MPI全名是Message Passing Interface，它是一个标准，而不是一个实现，专门为进程间通信实现的。它的工作原理很简单，启动一组进程，在同一个通信域中的不同进程有不同的编号，可以给不同编号的进程分配不同的任务，最终实现整个任务。
MPI4PY就是python中MPI的实现。在python中有很多种方法实现多进程以及进程间通信，比如multiprocessing，但是multiprocessing进程间通信不够方便，mpi4py的效率更高一些。
mpi4py提供了点对点通信，点对面，面对点通信。点对点通信又包含阻塞和非阻塞等等，通信的内容包含python内置对象，也包含numpy数组等。

## mpi4py简单对象和方法介绍
MPI.COMM_WORLD是一个通信域，在这个通信域中有不同的进程，每个进程的编号以及进程的数量都可以通过这个通信域获得。具体看以下comm_world.py代码：
``` python
from mpi4py import MPI

# 获得多进程通信域
comm = MPI.COMM_WORLD
# 获得当前进程通信域中进程数量
size = comm.Get_size()
# 获得当前进程在通信域中的编号
rank = comm.Get_rank()
```
>>> mpiexec -np 3 python comm_world.py

## 点对点通信
### 阻塞通信
#### python对象
##### 简介
comm.send(data, dest, tag)
comm.recv(source, tag)
send和recv都是阻塞方法，即调用这个方法之后，等到该函数调用结束之后再返回。dest是目的process编号，source是发送的process编号。data是要发送的数据，需要是python的内置对象，即可以pickle的对象。

##### 代码示例
``` python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

if rank == 0:
    data = {'name': "mxx", "age": 23}
    comm.send(data, dest=1, tag=10)
    print("data has sent.")
else:
    data = comm.recv(source=0, tag=10)
    print("data has been receieved.")
```

#### numpy数组
##### 简介
comm.Send(data, dest, tag)
comm.Recv(source, tag)
Send和Recv都是阻塞方法，即调用这个方法之后，等到该函数调用结束之后再返回。dest是目的process编号，source是发送的process编号。data是要发送的数据，需要是numpy对象，和c语言的效率差不多。

##### 代码示例
``` python
from mpi4py import MPI

comm = MPI.COMM_WORLD

rank = comm.Get_rank()
size = comm.Get_size()

if rank == 0:
    data = {'name': "mxx", "age": 23}
    comm.isend(data, dest=1, tag=10)
    print("data has sent.")
else:
    data = comm.irecv(source=0, tag=10)
    print("data has been receieved.")

```

### 非阻塞通信
#### 简介
comm.isend(data, dest, tag)
comm.irecv(source, tag)
isend和irecv都是非阻塞方法，即调用这个方法之后，调用该函数之后立即返回，无需等待它执行结束。dest是目的process编号，source是发送的process编号。data要是python对象，可以被pickle处理的。

#### 代码示例
``` python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD

rank = comm.Get_rank()
size = comm.Get_size()


if rank == 0:
    data = np.ones((3, 4), dtype='i')
    comm.Send([data, MPI.INT], dest=1, tag=10)
    print("data has sent.")
else:
    data = np.empty((3, 4), dtype='i')
    data = comm.Recv([data, MPI.INT], source=0, tag=10)
    print("data has been receieved.")

```

## 组通信
### bcast
#### 简介
将一个process中的数据发送给所有在通信池中的process。
comm.bcast(data, dest, tag)

#### 代码示例
``` python
import mpi4py
from mpi4py import MPI

comm = MPI.COMM_WORLD
size = comm.Get_size()
rank = comm.Get_rank()

if rank == 1:
    data = {"name": "mxx", "age": 23}
    print("data bcast to others")
else:
    data = None

data = comm.bcast(data, root=1)
print("process {} has received data".format(rank))

```

### scatter
#### 简介
将一个process的数据拆分成n份，发送给所有在通信池中的process每个一份，和bcast的区别在于，bcast发送的数据对于每一个process都是一样的，而scatter是将一份数据拆分成n份分别发送给每个process。
comm.scatter(data, dest, tag)

#### 代码示例
``` python
import mpi4py
from mpi4py import MPI

comm = MPI.COMM_WORLD
size = comm.Get_size()
rank = comm.Get_rank()

recv_data = None
if rank == 1:
    send_data = range(size) 
    print("data bcast to others")
else:
    send_data = None

recv_data = comm.scatter(send_data, root=1)
print("process {} has received data {}".format(rank, recv_data))

```

### gather
#### 简介
和comm.bcast相反，将每个process中的数据收集到一个process中。
comm.gather(data, dest, tag)

#### 代码示例
``` python
import mpi4py
from mpi4py import MPI


comm = MPI.COMM_WORLD
size = comm.Get_size()
rank = comm.Get_rank()

send_data = rank
print("process {} send data {} to root.".format(rank, send_data))

recv_data = comm.gather(send_data, root=9)
if rank == 9:
    print("process {} gather all data {} to others.".format(rank, recv_data))

```

## 参考文献
1.https://zhuanlan.zhihu.com/p/25332041
2.https://www.jianshu.com/p/f497f3a5855f

