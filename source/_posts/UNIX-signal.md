---
title: UNIX signal
date: 2019-12-04 15:22:47
tags:
 - UNIX
 - signal
categories: UNIX
---


## 概述
Signals是软件终端。它提供了一种处理异步事件的方法。

## 什么是signal
每一个signal都有一个名字，它们以三个字符`SIG`开头。例如：
- `SIGABRT`是abort signal，调用abort函数时产生这种signal
- `SIGALRM`是闹钟signal，由alarm函数设置的定时器超时后产生这种signal

在头文件`<singal.h>`中，signal name都被定义成正整数常量。如果内核包含对用户级应用程序有意义的文件，被认为是一种不好的形式，如果应用程序和内核需要使用同一个定义，那么就将有关信息放在内核头文件中，然后用户级头文件再包含该内核头文件。比如Linux 3.2.0将signal定义在`<bits/signum.h>`中。

很多条件可以产生signal：
- 用户按一些键
- 硬件异常
- 调用`kill(2)`函数
- 调用`kill(1)`命令，它是`kill(2)`的接口
- 检测到某种软件条件已经发生

常见的signal可以分为以下几类：
- 程序出错signals，用于report程序错误
- Termination singals，用于中断或者终止程序
- Alarm signals,
- 异步I/O signals
- Joc control signals
- 操作错误signal
- miscellaneous signals
- signal messages

关于具体的每个signal的介绍，可以看书，看文档`man 7 signal`，或者查看另一篇文章[UNIX signals]()。

在某个signal出现后，可以按照以下三种方式之一进行signal处理：
1. 忽略singal。有两个signal不能被忽略：`SIGKILL`和`SIGTSTP`，它们向内核或者root用户踢欧冠呢了停止或者终止进程的可靠方法。还有某些硬件signal，除零或者非法内存引用，进程的行为是未定义的。
2. 捕获signal。通过内核接收到某个signal后，调用相应的用户函数。
3. 执行系统默认动作。对于大多数系统，默认动作是终止进程的执行。


## 函数`signal`
`signal`是ISO C定义的。但是ISO C不涉及多进程，进程组，终端I/O等概念。所以它对signal的定义非常含糊，对于UNIX的用处很小。`signal`的实现在不同UNIX版本上是不同的，最好使用`sigaction`函数代替`signal`。

### `signal`定义
``` c
#include <signal.h>

typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```

### `signal`性质
1. `signal`函数有两个参数和一个返回值。第一个参数`signum`是整形，表示一个signal，第二个参数handler是函数指针，返回值也是一个函数指针。
2. `handler`的值是常量`SIG_IGN`，`SIG_DFL`或者一个函数的地址。分别表示忽略该信号，执行默认动作，或者调用相应的函数。
3. **`signal`函数的返回值是指向之前的信号处理程序的函数指针。**所以，对于signal来说，只有改变信号的处理方式才能确定信号的当前处理方式。

###  `exec`和`fork`
当使用`exec`执行一个程序时，所有signal的状态都是系统默认或者忽略。`exec`函数将原先设置为要捕获的signal更改为默认动作，其他signal的状态不变。比如一个进程原先要捕获的signal，执行一个新程序后就不再catch了，因为signal catch函数的地址可能在执行的新程序文件中无意义了。

而fork因为复制了父进程的内存映像，所以信号捕捉函数的地址在子进程中是有意义的，子进程继承了父进程的信号处理方式。

## 不可靠的`signal`
之前一些版本的signal是不可靠的，不可靠说的是：
1. signal会丢失。比如一个signal发生了，但是进程却不知道。
2. 进程对signal的控制能力很差。进程只能catch或者ignore signal，并不能阻塞signal。
3. 进程每次接到signal对其进行处理时，然后将signal重置为默认值。
4. 进程不希望发生某种signal时，不能关闭它，只能ignore它。

## 中断的系统调用
如果进程在执行一个**低速系统调用**而**阻塞**期间捕捉到一个signal，这个系统调用就会被中断不再继续执行（不再阻塞），返回出错，errno设置为EINTR。
什么是低速系统调用？系统调用被分为两类，一类是低速系统调用，另一类是其他系统调用。低速系统调用是指**可能会使进程永远阻塞**的一类系统调用，比如：
1. 如果某些类型文件（管道，终端设备和网络设备）的数据不存在，则读操作可能会使调用者永远阻塞。
2. pause和wait函数等。
3. ...

注意，与磁盘I/O有关的系统调用大多数时候总是会很快返回。

## 可重入函数
signal发生的时间是任意的，进程正在执行的正常指令可能会被信号处理程序中断，会对进程造成破坏。
SUS说明了在信号处理程序中保证调用安全的函数，这些函数是可重入的，被称为异步信号安全的(async-signal safe)。如下所示是异步信号安全的函数：
![re](re.png)
其余的大多数函数是不可重入的，因为它们可能满足以下条件：
1. 使用静态数据结构；
2. 调用`malloc`或者`free`；
3. 是标准I/O函数。

因为每个线程只有一个errno变量，所以信号处理程序可能会修改它的原值。因此，在调用可重入函数之前，应该先保存errno，在调用后恢复errno。

## SIGCLD语义

## 可靠signal

## 函数
ISO C并不涉及多进程，所以它不能定义以进程ID为参数的函数。

### `kill`和`raise`
kill向参数pid指定的进程或者进程组发送一个signal。
raise向调用者发送一个signal，ISO C中没有线程，POSIX.1扩展了raise可以处理多线程。单线程程序中等价于`kill(getpid(), sig)`。在多线程程序中等价于`pthread_kill(pthread_self(), sig)`。

### `alarm`和`pause`
alarm不阻塞，当某个时刻到达时，内核会产生一个SIGALRM信号。
pause使调用进程挂起，直到捕捉到任意一个信号，执行相应的信号处理程序，并从其返回时，pause才返回。

## 参考文献
1.《APUE》第三版
2.https://www.gnu.org/software/libc/manual/html_node/Standard-Signals.html#Standard-Signals

