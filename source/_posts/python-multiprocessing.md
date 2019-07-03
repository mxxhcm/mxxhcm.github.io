---
title: python multiprocessing 
date: 2019-04-23 15:46:14
tags:
- python
- Pool
- Process
- multiprocessing
categories: python
---

## Pool
### 简介
指定占用的CPU核数，进程的个数可以多于CPU的核数，Pool会负责调用。如果CPU核数小于进程数，一般遵循FIFO的原则进行调用。

### 使用流程
1. 创建Pool进程池，指定cpu核数
pool = Pool(cpu_core) 
2. 使用apply_async添加进程
processes = [p1, p2, p3]
for p in processes:
    pool.apply_async(p, args=())
3. 关闭进程池
pool.close()
4. 等待所有进程执行完毕
pool.join()


### 代码示例
[代码地址]()
``` python
from multiprocessing import Pool
import random
import time


def p1():
    print("A")
    # print("Run task %s (%s)" % (name, os.getpid()))
    start = time.time()
    print("start time", start)
    time.sleep(random.random()*3)
    end = time.time()
    print("task A runs %0.2f seconds." % (end-start))

def p2():
    print("B")
    # print("Run task %s (%s)" % (name, os.getpid()))
    start = time.time()
    print("start time", start)
    time.sleep(random.random()*3)
    end = time.time()
    print("task B runs %0.2f seconds." % (end-start))

def p3():
    print("C")
    # print("Run task %s (%s)" % (name, os.getpid()))
    start = time.time()
    print("start time", start)
    time.sleep(random.random()*3)
    end = time.time()
    print("task C runs %0.2f seconds." % (end-start))


if __name__ == "__main__":
    cpu_core = 2
    pool = Pool(cpu_core) 
    p = [p1, p2, p3]
    for h in p:
        pool.apply_async(h, args=())
    pool.close()
    pool.join()
    print("Done.")
```

## Process
### 简介
每个进程占用一个CPU核，进程核
### 代码示例
[代码地址]()
``` python
from multiprocessing import Process
import os,random,time


def proc1(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))

def proc2(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))

if __name__ == '__main__':
    # 1.Process
    print("Parent process %s" % os.getpid())
    p1 = Process(target=proc1, args=('p1',))
    p2 = Process(target=proc2, args=('p2',))
    print("child processes will start.")
    p1.start()
    p2.start()
    # 上面两行代码意思是p1.start()有返回值时，开始执行p2.start()。p1.start()有返回值并不是说p1执行完了
    p1.join()
    p2.join()
    # 上面两行代码中，p1.join()执行完之后才会执行p2.join()。所以只有p1执行完之后，p2才能尝试结束。。
    #  The interpreter will, however, wait until P1 finishes before attempting to wait for P2 to finish.
    print("child processes end.")
```

## Pool vs Process
1. Pool会负责对cpu进行调度，即tasks数量可以远大于process数量，一个process占用一个cpu核。而Process的task必须小于cpu核数，每个cpu核只能运行一个task。


## join方法
用来阻塞当前进程，直到该进程执行完毕，再继续执行后续代码。如下示例[代码](https://github.com/mxxhcm/myown_code/blob/master/tools/process_thread/multiprocessing_join.py)
``` python
from multiprocessing import Process,Pool
import os,random,time


def run_proc(name):
    time.sleep(2)
    print("Run child process %s (%s)" % (name,os.getpid()))


if __name__ == '__main__':
    print("Parent process %s" % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print("child process will start.")
    p.start()
    p.join()
    print("child process end.")
 
    print("Parent process %s" % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print("child process will start.")
    p.start()
    # p.join()
    print("child process end.")
```
输出结果：
> Parent process 7092
child process will start.
Run child process test (7093)
child process end.
Parent process 7092
child process will start.
child process end.
Run child process test (7094)

可以看出来，调用join()函数的时候，会等子进程执行完之后再继续执行；而不使用join()函数的话，在子进程开始执行的时候，就会继续向后执行了。


## 参考文献
1.https://www.cnblogs.com/lipijin/p/3709903.html
2.https://www.ellicium.com/python-multiprocessing-pool-process/
