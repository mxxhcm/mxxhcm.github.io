---
title: python multiprocessing 
date: 2019-04-23 15:46:14
tags:
- python
- multiprocessing
categories: python
---

## python 多线程 
### join方法
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
