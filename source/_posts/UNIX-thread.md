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

### 

### 总结
`pthread_create`,`pthread_exit`和`pthread_join`它们的void *类型的参数可以传递的值不止一个，还可以传递结构的地址，但是需要注意的是，这个结构所在的内存在调用者完成调用之后必须还是有效的（如果是当前线程在栈上分配的内存的话，返回值这个地址就无效了）。


## 参考文献
1.
