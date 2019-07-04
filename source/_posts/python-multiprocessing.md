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

## multiprocessing
### 概述
方法| 并行|是否直接阻塞|目标函数|函数返回值
--|--|--|--|--
mp.Pool.apply|否|是|只能有一个函数|函数返回值
mp.Pool.apply_async|是|否，调用join()进行阻塞|可以相同可以不同|返回AysncResult对象
mp.Pool.map|是|是|目标函数相同，参数不同|所有processes完成后直接返回有序结果
mp.Pool.map_async|是|否，调用join()阻塞|不知道。。|返回AysncResult对象
mp.Process|是|否|可以相同可以不同|无直接返回值

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

## Pool
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

#### Pool.apply vs apply_async
multiprocessing.Pool中也有类似的interface。Pool.apply和python内置的apply挺像的，只不过Pool.apply会在一个单独的process执行，并且该函数会阻塞直到进程调用结束，所以Pool.apply不能异步执行。可以使用apply_async使用多个workers并行处理。
Pool.apply_async和apply基本一样，只不过它会在调用后立即返回一个AsyncResult对象，不用等到进程结束再返回。然后使用get()方法获得函数调用的返回值，get()方法会阻塞直到process结束。也就是说Pool.apply(func, args, kwargs)和pool.apply_async(func, args, kwargs).get()等价。Pool.apply_async可以调用很多个不同的函数。
Pool.apply_async返回值是无序的。

#### Pool.map vs Pool.map_async
Pool.map应用于同一个函数的不同参数，它的返回值顺序和调用顺序是一致的。Pool.map(func, iterable)和Pool.map_async(func, iterable).get()是一样的。

#### Pool.map vs Pool.apply 
Pool.apply(f, args): f函数仅仅被process pool中的一个worker执行。
Pool.map(f, iterable): 将iterable分割成多个单独的task，就是相当于同一个函数，给定不同的参数，每一组是一个task，然后使用pool中所有的processes执行这些taskes。所以map也能实现并行处理，而且是有序结果。

#### Pool.map vs Pool.apply_async
Pool.map返回的结果是有序的；
Pool.apply_async返回的结果是无序的。
Pool.map处理相同的函数，不同的参数；
> pool.map() is a completely different kind of animal, because it distributes a bunch of arguments to the same function (asynchronously), across the pool processes, and then waits until all function calls have completed before returning the list of results.
Pool.apply_async处理不同的参数。

### retrieve return value
Pool.apply()会直接返回结果。
Pool.apply_async()会返回一个AsyncResult，然后使用get()方法获得结果。

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


## Process
### 简介
每个进程占用一个CPU核。

### retrieve结果
使用mp.Queue()或者mp.Pipe()等对象记录结果。Queue()不保证结果的顺序和task的执行顺序一致。

### 使用流程

### 代码示例
[代码地址]()


## Pool vs Process
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

## join方法
### 简介
用来阻塞当前进程，直到该进程执行完毕，再继续执行后续代码。
### 代码示例
[代码地址](https://github.com/mxxhcm/myown_code/blob/master/tools/py_process_thread/mp/mp_join.py)
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
