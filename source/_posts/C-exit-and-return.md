---
title: C/C++ exit and return
date: 2019-11-12 13:51:20
tags:
 - return
 - exit
 - C/C++
categories: C/C++
---

## 进程终止
总共有八种方式可以让进程终止，包括五种正常终止和三种异常终止，前五种是正常终止，后五种是异常终止：
1. 从`main`返回，相当于调用`exit`。
2. 调用`exit`，ISO C定义的，它的操作包括调用各个exit handler，处理所有标准I/O流。
3. 调用`_exit`或者`_Exit`，ISO C定义了`_Exit`，而POSIX.1说明了`_exit`。它的目的是提供一种无需运行exit handler或者信号处理程序而终止的方法。是否对标准I/O流进行flush，取决于实现。在UNIX中，`_Exit`和`_exit`是同义的，并不flush I/O流。
4. 最后一个线程从其启动例程返回
5. 最后一个线程调用`pthread_exit`
6. 调用`abort`
7. 接到一个`signal`
8. 最后一个线程对取消请求做出响应

不管进程以哪种方式终止，最后都会执行内核中的同一段代码，这段代码为相关进程关闭所有打开的文件描述符，释放它使用的内存。
为了让终止进程能够通知父进程它是如何终止的。对于3个终止函数(`exit`, `_exit`, `_Exit`)，将它们的`exit status`作为参数传递给函数。在异常终止的情况下，内核产生一个指示其异常终止原因的terminaiton status（终止状态）。在任意终止情况下，这个终止进程的父进程都能用`wait`或者`waitpid`函数获得它的终止状态。
**如果父进程在子进程之前终止**，那么对于父进程终止的所有进程，它们的父进程都变成`init`进程，终止状态返回到`init`进程。具体是怎么操作：对于一个即将终止的进程，内核检查所有活动进程，判断其中是否有待终止进程的子进程，如果有的话，将这些进程的父进程的ID改为`init`进程的ID 1。
**如果子进程在父进程之前终止**，那么父进程是无法获取它的终止状态的。内核为每一个终止进程保留了一部分信息，当终止进程的父进程调用`wait`或者`waitpid`时，可以获取这些信息，这些信息包含终止进程PID，进程的终止状态，进程占用的CPU时间总量。内核可以释放这些进程的内存，关闭打开的文件。如果一个进程终止了，但是它的父进程没有等待它，它被称为一个zombie（僵尸）进程。如果一个长期运行的进程，`fork`了很多子进程，除非父进程等到取得子进程的终止状态，要不它们就会变成僵尸进程。**当父进程结束时，僵尸进程就会结束？？？**
`init`的子进程，不会变成僵尸进程，因为`init`进程被编写成无论何时只要有一个子进程终止，`init`就会调用一个`wait`函数获得其终止状态。

### `exit`和`_Exit`函数
#### C11标准定义
`exit`定义在`<stdlib.h>`头文件中
> void exit( int exit_code ); (until C11)
> _Noreturn void exit( int exit_code );(since C11)
> Causes normal program termination to occur.
> Several cleanup steps are performed:
- functions passed to atexit are called, in reverse order of registration（调用atexit注册的函数）
- all C streams are flushed and closed （冲洗C的缓冲区，不是不关闭流吗？？？）
- files created by tmpfile are removed  (删除临时文件）
- control is returned to the host environment. If exit_code is zero or EXIT_SUCCESS, an implementation-defined status, indicating successful termination is returned. If exit_code is EXIT_FAILURE, an implementation-defined status, indicating unsuccessful termination is returned. In other cases implementation-defined status value is returned.（将控制权返还给操作系统。）

`-Exit`定义在`<stdlib.h>`头文件中
> void _Exit( int exit_code ); (since C99) (until C11)
> _Noreturn void _Exit( int exit_code );(since C11)
> Causes normal program termination to occur without completely cleaning the resources.
> Functions passed to at_quick_exit() or atexit() are not called. Whether open streams with unwritten buffered data are flushed, open streams are closed, or temporary files are removed is implementation-defined. （不调用atexit注册的函数，是否冲洗缓冲区，关闭打开的stream和删除临时文件是由实现定义的，UNIX都不做这些操作）
> If exit_code is 0 or EXIT_SUCCESS, an implementation-defined status indicating successful termination is returned to the host environment. If exit_code is EXIT_FAILURE, an implementation-defined status, indicating unsuccessful termination, is returned. In other cases an implementation-defined status value is returned.

#### 性质
1. `exit`和`_Exit`是ISO C的内容，而`_exit`是POSIX.1的内容。
2. 它们都用于正常终止一个程序，`_Exit`和`_exit`立刻进入内核，`_Exit`和`_exit`是否冲洗缓冲区是由实现定义的，UNIX上选择不冲洗。而`exit`先执行一些清理操作，然后返回内核，`exit`函数首先调用`atexit`函数登记的终止处理程序，然后冲洗标准I/O流，现代的`exit`实现都不会关闭标准I/O流，之前的一些实现还会关闭标准I/O流，这在调用`vfork`的时候可能会出现问题，还会删除临时文件。
3. 三个退出函数都需要一个整形的参数，被称为exit status。
4. 如果满足以下条件：
    - 调用这三个函数不带终止状态
    - `main`执行了一个不带返回值的`return`语句
    - `main`没有声明返回类型为整形，进程的终止状态是未定义的。
那么这个进程的终止状态是未定义的。
5. `main`返回返回一个整型值和用该值调用`exit`是等价的。对于某些C编译器和UNIX lint(1)程序来说，会产生警告信息，因为这些编译器并不了解`main`中的`return`和`exit`的作用是相同的。避开这种警告信息的一种方法是在`main`中使用`return`而不是`exit`，这样做的结果是UNIX grep命令无法找出程序中所有的`exit`调用。另一个方法是将`main`声明为`void`而不是`int`，然后调用`exit`，但是这不并不是标准，ISO C和POSIX.1定义`main`的返回值应当是带符号整形。

#### 函数原型
``` c
#include <stdlib.h>

void exit(int status);
void _Exit(int status);

#include <unistd.h>

void _exit(int status);
```

### `atexit`
#### C11标准定义
> Registers the function pointed to by func to be called on normal program termination (via exit() or returning from main()). The functions will be called in reverse order they were registered, i.e. the function registered last will be executed first.
> The same function may be registered more than once.
> atexit is thread-safe: calling the function from several threads does not induce a data race.
> The implementation is guaranteed to support the registration of at least 32 functions. The exact limit is implementation-defined.

#### 性质
每个进程可以通过`atexit` register至多32个由`exit`自动调用的函数，这些函数被称为exit handler（终止处理程序）。```c
#include <stdlib.h>

int atexit(void (*function)(void));
```
1. `atexit`的参数是一个函数地址，不会有返回值
2. `exit`调用`atexit` register的程序的顺序和使用`atexit`进行register的顺序相反。
3. ISO C和POSIX.1标准规定，`exit`首先调用各个exit handler，然后使用`fclose`关闭所有标准I/O流。
4. POSIX.1对ISO C进行了扩展，如果程序调用了任何`exec`函数，清除exit handler。
5. 内核执行一个程序的唯一方法是调用一个`exec`函数。进程自愿终止的唯一办法是显式或者隐式的（通过`exit`）调用`_exit`和`_Exit`。


## `return`


## `return`和`exit`
1. `return`是C/C++语言的关键字，是语言级别的；而`exit()`是一个函数，它是对系统调用`_exit()`的封装，是系统调用层次的。
2. `return`结束一个函数的执行；而`exit()`结束一个进程，删除进程使用的内存空间，并且将应用程序的一个状态返回给OS，这个状态标识了进程的运行信息。
3. `exit(0)`表示正常运行程序并退出程序，`exit(1)`表示非正常运行导致退出程序；`return 0`和`retrun 1`能够起类似的作用。
4. 对于我们自定的函数，可以return给操作系统，交给相关的处理程序调用exit或者程序自身直接调用exit。

### C++中的区别
在C++中，退出程序时，`exit`并不会调用局部非静态对象的析构函数，而`return`会调用局部非静态对象的析构函数。


## 参考文献
1.https://stackoverflow.com/questions/461449/return-statement-vs-exit-in-main
2.https://www.geeksforgeeks.org/return-statement-vs-exit-in-main/
3.https://www.zhihu.com/question/26591968/answer/33839473
4.https://www.zhihu.com/question/26591968/answer/33330774
