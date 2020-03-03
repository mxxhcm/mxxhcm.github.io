---
title: UNIX thread
date: 2020-01-04 19:11:28
tags:
 - UNIX
 - 线程
categories: UNIX
---

## 线程和进程的区别和联系
每个线程都包含有执行环境所必须的信息，包括进程中标识线程的线程ID，一组寄存器值，栈，调度优先级和策略，signal屏蔽字，errno变量和线程私有数据。
一个进程的所有信息对该进程的所有线程都是共享的，包括可执行程序的代码，程序的全局内存，堆内存，栈和文件描述符。

## 线程ID
线程ID只有在它所在的上下文才有意义。线程ID用`pthread_t`数据类型表示，实现的时候用一个结构体表示，所以不能把它当做整数处理。
在`pthread_t`的具体实现上，Linux使用无符号长整形表示（这个值看起来也像指针，但是不是指针），Solaris用无符号整形表示，FreeBSD和Mac OS X用指向pthread结构的指针表示。 

## 创建线程
使用`pthread_create`进行。如果失败，返回错误码，而不是设置errno。

## 线程终止的方式
如果一个线程调用了`exit`, `Exit`或者`_Exit`，那么整个进程都会终止。
或者信号的默认动作是终止进程，那么发送到线程的信号就会终止整个进程。
单个线程可以通过三种方式退出。
1. 线程从启动例程返回。
2. 被同一进程中的其他线程cancel。
3. 调用`pthread_exit`

## 线程同步
### 互斥量
互斥量有两种状态，要不加锁，要不就是不加锁。
一次只有一个线程能够加锁。

### 避免死锁
### 超时锁
### 读写锁
读写锁有三种状态：
读锁，写锁，不加锁。
一次只有一个线程可以占有写锁，但是多个线程可以同时占有读锁。读锁和写锁不能同时存在。

### 带有超时的读写锁筛选条件
筛选条件

### 条件变量
条件变量和互斥量的区别[2]。
mutex只是让我们阻塞直到lock可用。
而condition variable让我们阻塞到某个用户定义的条件可用。

### 自旋锁
mutex在阻塞的时候是sleep。
而自旋锁在阻塞的时候是占用cpu。

### 屏障
pthread_join就是一种barrier。
只不过pthread_barrier_wait() wait的是所有count个线程都要调用pthread_barrier_wait()。
而pthread_join() wait的是线程的返回值。
如果其他线程没有结束，主线程中没有调用pthread_join()获得这些线程的返回值。则主线程可能结束的比这些线程早，从而使得这些进程尚未完成就退出了（因为主进程会return或者调用exit)。

pthread_wait()的返回值，总共有count个线程调用，只有一个返回`PTHREAD_BARRIER_SERIAL_THREAD`，其他都返回0。

## 线程限制

## 线程属性

## 同步属性

## 重入

## 线程和特定数据

## 取消选项

## 线程和fork

## 线程和IO

## 函数
### `pthread_self`
获得当前进程ID

### `pthread_equal`
判断两个进程ID是否相等，返回值是0表示不相等

### `pthread_create`
创建线程，返回值是0表示正常退出。

### `pthread_exit`
终止当前线程，设置的retval可以被`pthread_join`得到。``` c
void pthread_exit(void *retval);
```
此外，可以直接return一个void *类型的值结束当前线程。

### `pthread_join`
`pthread_join`可以通过`pthread_`获得一个终止线程（通过`pthread_exit`结束或者return的void*的值）的exit code。```c
int pthread_join(pthread_t thread, void **retval);
```

### `pthread_cancel`
给一个线程发送一个取消请求。``` c
int pthread_cancel(pthread_t thread);
```

### `pthread_cleanup_push`和`pthread_cleanup_pop`
线程处理程序，和`atexit`类似。它们都操作的是thread-cancellation clearn-up handlers的调用栈。当一个thread被cancelled时，自动执行一个clean-up handler函数。
`pthread_clean_push`将一个清理函数`routine`push到clean-up hanlers栈的最上面，当routine随后被调用的时候，传递arg参数给它。
`pthread_clean_pop`从clean-up栈的栈顶将routine移除，如果execute位不为0的话执行它。

在以下情况下，一个cancellation clean-up handler从栈中弹出，并执行：
1. 一个thread被canceled，弹出所有clean-up handlers。
2. 一个thread被调用`pthread_exit`终止，弹出所有的clearn-up handlers。
3. 一个thread调用`pthread_clean_push`，并且参数excute为非0值，弹出栈定的clean-up handlers。


### 总结
`pthread_create`,`pthread_exit`和`pthread_join`它们的void *类型的参数可以传递的值不止一个，还可以传递结构的地址，但是需要注意的是，这个结构所在的内存在调用者完成调用之后必须还是有效的（如果是当前线程在栈上分配的内存的话，返回值这个地址就无效了）。


## 参考文献
1.《APUE》第三版
2.https://stackoverflow.com/questions/4742196/advantages-of-using-condition-variables-over-mutex
