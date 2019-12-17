---
title: UNIX system call vs library call
date: 2019-11-12 13:01:01
tags:
 - 操作系统
 - UNIX
categories: UNIX
---

## 系统调用(system call)和库函数(library function)
### 系统调用
所有的操作系统都提供多种服务的入口点，通过这些入口点向内核请求服务，这些入口点被称为系统调用(system call)。系统调用处于kernel mode，一些任务只能在kernel mode运行。比如和硬件的交互，系统调用使得用户mode的进程可以通过系统调用进入kernel mode，从而实现和硬件的交互。。
系统调用接口可以在man的第二部分查看，它是用C语言定义的，与具体系统如何调用一个系统调用的实现技术无关。这些和早期的操作系统按照传统方式用机器的汇编语言定义内核入口点。
UNIX使用的方法是为每个系统调用在标准C库中设置一个同名函数。用户进程使用标准C调用相应的函数，这些函数又根据系统调用调用相应的内核服务。

### 库函数
库函数可以在man手册的第三部分查看，第三部分定义了程序员可以使用的通用库函数。库函数可以调用系统调用，也可以不调用系统调用，比如`read`函数会调用系统调用，而`atoi`等并不使用任何系统调用。

### 系统调用和库函数之间的关系
1. 从实现角度来看，系统调用和库函数有着根本的区别，系统调用处于内核mode，库函数属于用户mode。
2. 从用户应用角度考虑，可以把系统调用看做C函数，使用系统调用还是库函数不重要，它们都是为应用程序提供服务的。
3. C函数只是系统调用和库函数的一种实现，系统调用和库函数都可以以其他方式实现。
4. 系统调用通常只是提供一种最小接口，而库函数实现更复杂的功能。
5. 库函数可以被替换，但是系统调用通常是不能替换的。
6. 库函数可以调用系统调用，也可以不调用系统调用。
7. 应用程序既可以调用库函数也可以调用系统调用。
8. 进程控制系统调用(fork, exec和wait)等通常由应用程序直接调用。为了简化一些常见情况，UNIX也提供了一些库函数，如system和popen。
9. 库函数链接到用户程序，在user space执行，而syste call没有链接到用户程序，在kernel space执行
10. 库函数的执行时间被计算为user level time，而system call的执行事件算作system time的一部分。
11. 库函数可以简单的进行debug，而系统调用不能debug，因为它们被kernel执行。

对于第4条，可以考虑以下例子：
sbrk(2)是分配存储空间的UNIX系统调用，它按照指定字节数增加或者减少进程地址空间。如何管理进程的地址空间由进程决定。
malloc(3)是公用函数库中的一个存储分配空间函数，它负责进行进程的存储地址管理。
我们可以自己实现一个malloc，但是它很有可能还要使用sbrk(2)。内核中的系统调用ssbrk是系统层面的空间分配，而库函数malloc是在用户层面进行操作。


## 参考文献
1.《APUE》第三版
2.https://www.thegeekstuff.com/2012/07/system-calls-library-functions/
3.https://stackoverflow.com/questions/29816791/what-is-the-difference-between-system-call-and-library-call
4.https://unix.stackexchange.com/questions/6931/what-is-the-difference-between-a-library-call-and-a-system-call-in-linux
5.https://unix.stackexchange.com/questions/57232/difference-between-system-calls-and-library-functions

