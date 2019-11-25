---
title: UNIX Process Environment
date: 2019-11-25 10:22:51
tags:
 - 进程
 - UNIX
categories: UNIX
---

## 概述
这一部分介绍的是进程运行的环境。主要包括进程执行时，`main`函数是如何被调用的，命令行参数如何传递给进程的，进程的存储空间结构，如何分配存储空间，环境变量的使用，以及进程是怎么终止的。

## `main`函数和`argc`, `argv`
C语言总是从`main`函数开始执行，C语言中`main`有两个原型：``` c
int main(void);
int main(int argc, char *argv[]);
```
其中`argc`是命令行参数的个数，`argv`是一个指针数组，ISO C和POSIX.1都要求`argv[argc]`设置为`NULL`，所以可以用它作为参数处理的循环终止条件。关于更多指针的信息可以查看[]()。
当内核执行C程序（使用一个`exec`函数）时，在调用`main`函数之前设置一个特殊的启动例程。可执行程序文件将这个启动例程指定为程序的起始地址，这是由link editor设置的，它会被C编译器调用。启动例程从内核命令获得命令行参数和环境变量值，然后为调用`main`函数做好准备。

关于更多C和C++中`main`的介绍，可以查看[C/C++ main argc argv](https://mxxhcm.github.io/2019/11/12/C-CPP-main-argc-argv/)。

## 全局变量`environ`
每个C程序都会接收到一个environment list，和`argv`一样，它是一个指针数组。每个指针指向一个以`null`结束的C字符串的地址，这个指针数组的地址存放在全局变量`environ`中：``` c
extern char **environ;
```
在历史上，UNIX大多支持三个参数的`main`函数，第三个参数就是environment list：```c
int main(int argc, char *argv[], char *envp[]);
```
但是ISO C规定`main`只能有两个参数，POSIX.1也就规定使用全局变量`environ`而不是第三个参数。如果要查看所有的环境变量时，使用`environ`，而访问某个特定的环境变量时，使用`getenv`和`setenv`。


## 进程终止
总共有八种方式可以让进程终止，包括五种正常和三种异常，前五种是正常终止，后五种是异常终止：
1. 从`main`返回
2. 调用`exit`
3. 调用`_exit`或者`_Exit`
4. 最后一个线程从其启动例程返回
5. 最后一个线程调用`pthread_exit`
6. 调用`abort`
7. 接到一个`signal`
8. 最后一个线程对取消请求做出响应

### 退出函数
#### 函数原型
``` c
#include <stdlib.h>

void exit(int status);
void _Exit(int status);

#include <unistd.h>

void _exit(int status);
```

#### 性质
1. `exit`和`_Exit`是ISO C的内容，而`_exit`是POSIX.1的内容
2. `exit`函数总是执行一个标准I/O库的关闭操作，对于所有打开的流调用`fclose`函数，所有带有未写缓冲的标准I/O流被flush。
3. 三个退出函数都需要一个整形的参数，被称为exit status。
4. 如果满足以下条件：
    - 调用这三个函数不带终止状态
    - `main`执行了一个不带返回值的`return`语句
    - `main`没有声明返回类型为整形，进程的终止状态是未定义的。
那么这个进程的终止状态是未定义的。
5. `main`返回返回一个整型值和用该值调用`exit`是等价的。对于某些C编译器和UNIX lint(1)程序来说，会产生警告信息，因为这些编译器并不了解`main`中的`return`和`exit`的作用是相同的。避开这种警告信息的一种方法是在`main`中使用`return`而不是`exit`，这样做的结果是UNIX grep命令无法找出程序中所有的`exit`调用。另一个方法是将`main`声明为`void`而不是`int`，然后调用`exit`，但是这不并不是标准，ISO C和POSIX.1定义`main`的返回值应当是带符号整形。
关于`return`和`exit`的区别，更多可以查看[]()。

### `atexit`



## C程序的存储空间布局
更多关于C程序存储空间布局可以查看[]()。

## 共享库

## 存储空间分配

## 环境变量

## `setjmp`和`longjmp`

## `getrlimit`和`setrlimit`

## 参考文献
1. 《APUE》第三版

