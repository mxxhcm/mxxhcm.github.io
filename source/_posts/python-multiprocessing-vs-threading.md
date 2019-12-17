---
title: python multiprocessing 
date: 2019-04-23 15:46:14
tags:
 - python
categories: python
---

## multiprocessing vs multithread
多个threads可以在一个process中。同一个process中的所有threads共享相同的memory。而不同的processes有不同的memory areas，每一个都有自己的variables，进程之间为了通信，需要使用其他的channels，比如files, pipes和sockets等。thread比process更容易创建和管理，thread之间的交流比processes之间的交流更快。
这一节首先介绍一些GIL，然后介绍两个python的package，一个是threading，一个是multiprocessing。threading主要提供了多线程的实现。multiprocessing 主要提供了多进程的实现，当然也有多线程实现。

## GIL
thread有一个东西，叫做GIL(Global Interpreter Lock)，阻止同一个process中不同threads的同时运行，所以python多线程并不是多线程。举个例子，如果你有8个cores，使用8个threads，CPU的利用率不会达到800%，也不会快8倍。它会使用100%CPU，速度和原来相同，甚至会更慢，因为需要对多个threads进行调度。当然，有一些例外，如果大量的计算不是使用python运行的，而是使用一些自定义的C code进行GIL handling，就会得到你想要的性能。对于网络服务器或者GUI应用来说，大部分的事件都在等待，而不是在计算，这个时候就可以使用多个thread，相当于把他们都放在后台运行，而不需要终止相应的主线程。
如果想用纯python代码进行大量的CPU计算，使用threads并不能起到什么作用。使用process就没有GIL的问题，每个process有自己的GIL。这个时候需要在多线程和多进程之间做个权衡，因为进程之间的通信比线程之间通信的代价大得多。

## CPython的GIL实现

CPython 2.7中GIL是这样一行代码：
``` c
static PyThread_type_lock interpreter_lock = 0; /* This is the GIL */
 
from multiprocessing import Process, Queue
import os,random,time


def proc1(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))
    return "return A"

def proc2(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))
    return "return B"

def proc(length, output):
    result = "Hello! " + str(length) + "!"
    time.sleep(random.random())
    output.put(result)

if __name__ == '__main__':
    # 1. mp.Process
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


    # 2.获得mp.Process的返回值
    print("# 2.获得mp.Process的返回值")
    output = Queue()
    processes = [Process(target=proc, args=(x, output)) for x in range(4)]
    for p in processes:
        p.start()

    for p in processes:
        p.join()

    results = [output.get() for p in processes]
    print(results)

# https://stackoverflow.com/questions/31711378/python-multiprocessing-how-to-know-to-use-pool-or-process
```
在Unix类系统中，PyThread_type_lock是标准的C lock mutex_t的别名。它的初始化方式如下：
``` c
void
PyEval_InitThreads(void)
{
    interpreter_lock = PyThread_allocate_lock();
    PyThread_acquire_lock(interpreter_lock);
}
```
解释器中执行python的C代码必须持有这个lock。GIL的作用就是让你的程序足够简单：一个thread执行python代码，其他N个thread sleep或者等待I/O。或者可以等待threading.Lock或者其他同步操作。
那么什么时候threads进程切换呢？当一个thread 准备sleep或者进入等待I/O的时候，它释放GIL，其他thread请求GIL，执行相应的代码。这种任务叫做cooperative multitasking。还有一种是preemptive multitasking：在python2中一个thread不间断的执行1000个bytecode，或者python3中不间断的执行15 ms，然后放弃GIL让另一个thread运行。接下来举两个例子。

## cooperative multithread
在网络I/O中，具有很强的不确定性，当一个拥有GIL的thread请求网络I/O时，它释放GIL，这样子其他thread可以获得GIL继续执行，等到I/O完成时，该thread请求GIL继续执行。
``` python
def do_connect():
    s = socket.socket()
    s.connect(('python.org', 80))  # drop the GIL

for i in range(2):
    t = threading.Thread(target=do_connect)
    t.start()
```
在上面的例子中，同一时刻只能有一个拥有GIL的thread执行python代码，但是一旦拥有GIL的thread开始connect，它就drop GIL，另一个thread可以申请GIL。但是所有的threads都可以drop GIL，也就是多个thread可以一起并行的等待sockets连接。
具体python在connect socket的时候是怎么drop GIL的，我们可以看一下socketmodule的c代码：
``` c
/* s.connect((host, port)) method */
static PyObject *
sock_connect(PySocketSockObject *s, PyObject *addro)
{
    sock_addr_t addrbuf;
    int addrlen;
    int res;

    /* convert (host, port) tuple to C address */
    getsockaddrarg(s, addro, SAS2SA(&addrbuf), &addrlen);

    Py_BEGIN_ALLOW_THREADS
    res = connect(s->sock_fd, addr, addrlen);
    Py_END_ALLOW_THREADS

    /* error handling and so on .... */
}
```
其中Py_BEGIN_ALLOW_THREADS宏就是drop GIL，它的定义如下：
``` c
PyThread_release_lock(interpreter_lock);
```
同样，Py_END_ALLOW_THREADS宏是请求GIL。thread可以在这里block，等待GIL被释放，申请GIL继续执行。

## preemptive multithread
除了自动释放GIL外，还可以强制的释放GIL。python代码的执行有两步，第一步将python源代码编译成二进制的bytecode；第二步，python interpreter的main loop，一个叫做PyEval_EvalFrameEx()的函数，读取bytecode，并且一个一个的执行。
在多线程的模式下，interpreter强制周期性的drop GIL。如下所示，是thread判断是否释放GIl的代码：
``` c
for (;;) {
    if (--ticker < 0) {
        ticker = check_interval;
    
        /* Give another thread a chance */
        PyThread_release_lock(interpreter_lock);
    
        /* Other threads may run now */
    
        PyThread_acquire_lock(interpreter_lock, 1);
    }

    bytecode = *next_instr++;
    switch (bytecode) {
        /* execute the next instruction ... */ 
    }
}
```
默认设置下是1000个bytecode。所有的threads周期性的获取GIL，然后释放。在python3下，所有thread获得15ms的GIL，而不是1000个bytecode。

## python的thread safety 
但是，如果买票等之类的，必须保证操作的atomic，否则就会出现问题。对于sort() operation来说，它是atomic，所以无序担心。看下面一个code snippet
``` python
n = 0

def foo():
    global n
    n += 1
```
我们查看foo对应的bytecode：
``` python
import dis

print(dis.dis(foo))

#   7           0 LOAD_GLOBAL              0 (n)
#               2 LOAD_CONST               1 (1)
#               4 INPLACE_ADD
#               6 STORE_GLOBAL             0 (n)
#               8 LOAD_CONST               0 (None)
#              10 RETURN_VALUE
```
可以看出，foo有6个bytecode，如果在第三个bytecode处，强制释放了GIL锁，其他thread改了n的值，等到切回这个thread的时候，就会出错。。所以，为了保证不出问题，需要手动加一个lock，保证不会在这个时候释放GIL。
``` python
import threading

n = 0
lock = threading.Lock()

def foo():
    global n
    with lock:
        n += 1

```
当然，如果operation本身就是atomic的话，就不需要了。
``` python
l = [1, 3, 5, 2]

def foo():
    l.sort()
```

## threading 
threading是python多线程的一个package。

### threading.Thread

#### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/py_process_thread/threading_Thread.py)
``` python
import threading
import socket
import os

def do_connect(website):
    s = socket.socket()
    info = s.connect((website, 80))  # drop the GIL
    print(type(info))
    print(info)
    print(os.getpid())

websites = ['python.org', 'baidu.com']

job_list = []
for i in range(len(websites)):
    job_list.append(threading.Thread(target=do_connect, args=(websites[i],)))

for t in job_list:
    t.start()

# join表示阻塞，一直到当前任务完成为止，如果不加的话，就会立刻执行下面的print语句
for t in job_list:
    t.join()

print("Done")
```

### threading.Lock

#### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/py_process_thread/threading_Lock.py)
``` python
import time
import threading
import random

hhhh = 100
lock = threading.Lock()

def add_number():
    global hhhh
    for i in range(100):
        with lock:
            hhhh += 1
            print("add: ", hhhh)
            time.sleep(0.015)
    
def subtract_number():
    global hhhh
    for i in range(100):
        with lock:
            hhhh -= 1
            print("subtract:", hhhh)
            time.sleep(0.015)
 
job_list = []
job_list.append(threading.Thread(target=subtract_number, args=()))
job_list.append(threading.Thread(target=add_number, args=()))

for t in job_list:
    t.start()

for t in job_list:
    t.join()
 
print("Done")
```

## multiprocessing
### 概述
方法| 并行|是否直接阻塞|目标函数|函数返回值|适用场景
--|--|--|--|--
mp.Pool.apply|否|是|只能有一个函数|函数返回值
mp.Pool.apply_async|是|否，调用join()进行阻塞|可以相同可以不同|返回AysncResult对象
mp.Pool.map|是|是|目标函数相同，参数不同|所有processes完成后直接返回有序结果
mp.Pool.map_async|是|否，调用join()阻塞|不知道。。|返回AysncResult对象
mp.Process|是|否|可以相同可以不同|无直接返回值|适用于线程数量比较小

mp.Pool适用于线程数量远大于cpu数量，mp.Process适用于线程数量小于或者等于cpu数量的场景。
mp.Pool.apply   适用于非并行，调用apply()直接阻塞，process执行结束后直接返回结果。
mp.Pool.apply_async 适用于并行，异步执行，目标函数可以相同可以不同，返回AysncResult对象，因为AsyncResult对象是有序的，所以调用get得到的结果也是有序的。调用join()进行阻塞，调用get()方法获得返回结果，get()方法也是阻塞方法。
mp.Pool.map     适用于并行，异步，目标函数相同，参数不同。调用map()函数直接阻塞，等待所有processes完成后直接返回有序结果。
mp.Pool.map_async   也是调用join()和get()都能阻塞。
mp.Process  适用于并行，异步，目标函数可以相同可以不同，返回的结果需要借助mp.Queue()等工具，mp.Queue()存储的结果是无序的，mp.Manager()存储的结果是有序的。无序的结果可以使用特殊方法进行排序。


### 统计cpu数量
``` python
cpus = mp.cpu_count()
```
### 实现并行的几种常用方法
``` python
# 方式1
pool.apply_async
# 方式2
pool.map
# 方式3
mp.Process
```
### retrieve并行结果 
``` python
# 方式1
results_obj = [pool.apply_async(f, args=(x,)) for x in range(3)]
results = [result_obj.get() for result_obj in results_obj]

# 方式2
results = pool.map(f, range(7))

# 方式3
output = Queue()
pool.Process(target=f, args=(output))
```

## mp.Pool
### 简介
指定占用的CPU核数，进程的个数可以多于CPU的核数，Pool会负责调用。如果CPU核数小于进程数，一般遵循FIFO的原则进行调用。

### API
- Pool.apply, 
- Pool.apply_async, 
- Pool.map, 
- Pool.map_async。

#### python apply
在老版本的python中，调用具有任意参数的function要使用apply函数，
``` python
apply(f, args, kwargs)
```
甚至在2.7版本中还存在apply函数，但是基本上不怎么用了，3版本中已经没有了这种形式，现在都是直接使用函数名：
``` python
f(*args, **kwargs)
```

#### mp.Pool.apply vs mp.Pool.apply_async
multiprocessing.Pool中也有类似的interface。Pool.apply和python内置的apply挺像的，只不过Pool.apply会在一个单独的process执行，并且该函数会阻塞直到进程调用结束，所以Pool.apply不能异步执行。可以使用apply_async使用多个workers并行处理。
Pool.apply_async和apply基本一样，只不过它会在调用后立即返回一个AsyncResult对象，不用等到进程结束再返回。然后使用get()方法获得函数调用的返回值，get()方法会阻塞直到process结束。也就是说Pool.apply(func, args, kwargs)和pool.apply_async(func, args, kwargs).get()等价。Pool.apply_async可以调用很多个不同的函数。
Pool.apply_async返回值是无序的。

#### mp.Pool.map vs mp.Pool.map_async
Pool.map应用于同一个函数的不同参数，它的返回值顺序和调用顺序是一致的。Pool.map(func, iterable)和Pool.map_async(func, iterable).get()是一样的。

#### mp.Pool.map vs mp.Pool.apply 
Pool.apply(f, args): f函数仅仅被process pool中的一个worker执行。
Pool.map(f, iterable): 将iterable分割成多个单独的task，就是相当于同一个函数，给定不同的参数，每一组是一个task，然后使用pool中所有的processes执行这些taskes。所以map也能实现并行处理，而且是有序结果。

#### mp.Pool.map vs mp.Pool.apply_async
Pool.map返回的结果是有序的；
Pool.apply_async返回的结果是无序的。
Pool.map处理相同的函数，不同的参数；
> pool.map() is a completely different kind of animal, because it distributes a bunch of arguments to the same function (asynchronously), across the pool processes, and then waits until all function calls have completed before returning the list of results.
Pool.apply_async处理不同的参数。

### retrieve return value
Pool.apply()会直接返回结果。
Pool.apply_async()会返回一个AsyncResult，然后使用get()方法获得结果。

### 其他问题
pool.map传递多个参数，或者重复参数，使用他的另一个版本，pool.starmap()
如下示例，[代码地址]()
``` python
from multiprocessing import Pool
import time
import os
from itertools import repeat


def f(string, x):
    print(string)
    return x*x

if __name__ == "__main__":
    with Pool(processes=4) as pool:
        number = 10
        s = "hello"
        print(pool.starmap(f, zip(repeat(s), range(number))))
```

### 使用流程
1. 创建Pool进程池，指定cpu核数
pool = Pool(cpu_core) 
2. 使用apply_async添加进程
processes = [p1, p2, p3]
results = []
for p in processes:
    results.append(pool.apply_async(p, args=()))
3. 关闭进程池
pool.close()
4. 等待所有进程执行完毕
pool.join()
5. 访问结果
for res in results:
    print(res.get())


### 代码示例
[代码地址]()


## mp.Process
### 简介
每个进程占用一个CPU核。

### retrieve结果
使用mp.Queue()或者mp.Pipe()等对象记录结果。Queue()不保证结果的顺序和task的执行顺序一致。

### 使用流程

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/py_process_thread/mp/Process.py)
``` python
 
from multiprocessing import Process, Queue
import os,random,time


def proc1(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))
    return "return A"

def proc2(name):
    print("Run child process %s (%s)" % (name,os.getpid()))
    print(time.time())
    time.sleep(random.random())
    print("%s end" % (name))
    return "return B"

def proc(length, output):
    result = "Hello! " + str(length) + "!"
    time.sleep(random.random())
    output.put(result)

if __name__ == '__main__':
    # 1. mp.Process
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


    # 2.获得mp.Process的返回值
    print("# 2.获得mp.Process的返回值")
    output = Queue()
    processes = [Process(target=proc, args=(x, output)) for x in range(4)]
    for p in processes:
        p.start()

    for p in processes:
        p.join()

    results = [output.get() for p in processes]
    print(results)

# https://stackoverflow.com/questions/31711378/python-multiprocessing-how-to-know-to-use-pool-or-process
```


## mp.Pool vs mp.Process
1. Pool会负责对cpu进行调度，即tasks数量可以远大于worker数量，一个worker占用一个cpu核。而Process的task必须小于worker，每个worker只能运行一个task。
2. 如果执行多个task的时候，Process一定会使用多个seperate workes，但是对于Pool来说，可能会使用同一个worker去执行多个task。如下示例，p1和p2一定是两个wrokers运行两个process，而pool中，pool中有两个worker，foo可以是第一个worker也可以是第二个worker运行的process解决的，而bar也可以是这两个中任意一个worker解决的，这种情况发生在foo已经运行结束了，两个worker都是空闲的，给bar任意分配一个worker。
``` python
def foo():
    pass

def bar():
    pass

p1 = Process(target=foo, args=())
p2 = Process(target=bar, args=())

p1.start()
p2.start()
p1.join()
p2.join()

pool = Pool(processes=2)             
r1 = pool.apply_async(foo)
r2 = pool.apply_async(bar)
```

### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/tools/py_process_thread/mp/Pool_Process.py)
```  python
from multiprocessing import Process,Pool
import os,random,time


def run_proc(name):
    print("Run child process %s (%s)" % (name,os.getpid()))


def long_time_task(name):
    print("Run task %s (%s)" % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random()*3)
    end = time.time()
    print("task %s runs %0.2f seconds." % (name,end-start))


if __name__ == '__main__':
    # 1.Process
    print("Parent process %s" % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print("child process will start.")
    p.start()
    p.join()
    print("child process end.")

    # 2.Pool
    print("Paranet Process %s" % os.getpid())
    pool = Pool(4)
    for i in range(5):
        pool.apply_async(long_time_task,args=(i,))
    print("Waitting for done.")
    pool.close()    # 回收Pool
    pool.join()
    print("All subprocesses done")
```

## multiprocessing join方法
### 简介
用来阻塞当前进程，直到该进程执行完毕，再继续执行后续代码。

### 代码示例
[代码地址](https://github.com/mxxhcm/myown_code/blob/master/tools/py_process_thread/mp/join.py)
``` python
from multiprocessing import Process,Pool
import os,random,time


def run_proc(name):
    time.sleep(2)
    print("Run child process %s (%s)" % (name,os.getpid()))


if __name__ == '__main__':
    print("==========1. join==========")
    print("Parent process %s" % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print("child process will start.")
    p.start()
    p.join()
    print("child process end.")

 
    print("==========2. no join==========")
    print("Parent process %s" % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print("child process will start.")
    p.start()
    # p.join()
    print("child process end.")

```
可以看出来，调用join()函数的时候，会等子进程执行完之后再继续执行；而不使用join()函数的话，在子进程开始执行的时候，就会继续向后执行了。



## 参考文献
1.https://www.cnblogs.com/lipijin/p/3709903.html
2.https://www.ellicium.com/python-multiprocessing-pool-process/
3.https://stackoverflow.com/questions/8533318/multiprocessing-pool-when-to-use-apply-apply-async-or-map<mp Pool apply, apply_async, map用法>
4.https://stackoverflow.com/questions/31711378/python-multiprocessing-how-to-know-to-use-pool-or-process<mp Process和Pool.map获得不同目标函数process的结果，对mp.Process无序结果进行排序>
5.https://stackoverflow.com/questions/18176178/python-multiprocessing-process-or-pool-for-what-i-am-doing<mp Pool.apply_async, Process不同函数的多process>
6.https://stackoverflow.com/questions/10415028/how-can-i-recover-the-return-value-of-a-function-passed-to-multiprocessing-proce<获得传递给mp Process函数返回值的方法>
7.https://docs.python.org/3/library/multiprocessing.html#sharing-state-between-processes
8.https://sebastianraschka.com/Articles/2014_multiprocessing.html
9.https://opensource.com/article/17/4/grok-gil<GIL解释>
