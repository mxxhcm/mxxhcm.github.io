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
每个进程可以通过`atexit` register至多32个由`exit`自动调用的函数，这些函数被称为exit handler（终止处理程序）。```c
#include <stdlib.h>

int atexit(void (*function)(void));
```
1. `atexit`的参数是一个函数地址，不会有返回值
2. `exit`调用`atexit` register的程序的顺序和使用`atexit`进行register的顺序相反。
3. ISO C和POSIX.1标准规定，`exit`首先调用各个exit handler，然后使用`fclose`关闭所有标准I/O流。
4. POSIX.1对ISO C进行了扩展，如果程序调用了任何`exec`函数，清除exit handler。
5. 内核执行一个程序的唯一方法是调用一个`exec`函数。进程自愿终止的唯一办法是显式或者隐式的（通过`exit`）调用`_exit`和`_Exit`。


## C程序的存储空间布局
更多关于C程序存储空间布局可以查看[C/C++ program memory layout](https://mxxhcm.github.io/2019/10/19/C-program-memory-layout/)。

## 共享库
共享库使得可执行文件中不再需要包含公用的库函数，只需要在所有进程都可引用的存储区保存这种库例程的一个副本。程序第一次执行或者第一次调用某个库函数时，使用动态链接的方法将程序和共享库函数链接，这减少了每个可执行文件的长度，但是增加了一些时间运行开销。这种时间开销发生在程序第一次被执行时，或者每个共享库函数第一次被调用时。共享库的另一个优点是可以使用库函数的新版本代替老版本而无需对使用该库的程序重新链接和编辑。

## 内存空间分配
ISO C说明了三个用于memory allocation的函数，`malloc`, `calloc`和`realloc`，它们的原型如下：

### `malloc`, `calloc`和`realloc`原型
``` c
#include <stdlib.h>

void *malloc(size_t size);
void free(void *ptr);
void *calloc(size_t nmemb, size_t size);
void *realloc(void *ptr, size_t size);
void *reallocarray(void *ptr, size_t nmemb, size_t size);
```

### `malloc`, `calloc`和`realloc`属性
1. `malloc`，分配指定字节的内存空间，初始值不定。
2. `calloc`，为指定长度的固定数量的对象分配空间，每一个bit都被初始化为0。
3. `realloc`，增加或者减少已经分配的内存空间的大小。当这个大小增加时，可能需要将之前分配的空间中的数据移到另一个足够大的区域以便于增加大小，新增加的区域内的值是不确定的。
4. 这三个函数返回的指针一定是对齐的，保证它可以用于任何对象。比如`double`的要求最严格，需要从8的倍数的地址单元开始，这三个函数返回的地址一定满足这个要求。
5. 它们的返回类型都是`void*`，需要使用强制类型转换。
6. `realloc`函数可以增加或者减少之前分配的内存空间的大小。比如分配了一个固定大小的数组，后来发小它不够用了，可以使用`realloc`对它进行扩充，如果原有的存储后有足够的大小进行扩充，则可以在原存储区的位置上向高地址进行扩充，无需移动原有数组，返回和传入相同的指针。如果原来的内存空间后没有足够的空间，就重新分配一个足够大的内存空间，再将原有数据的内容复制过去，然后释放原来的内存空间，返回新的指针。
7. `realloc`传入的参数是存储区的新长度。如果传入的`ptr`参数是`NULL`指针，那就退化成了`malloc`。
8. `free`可以释放`ptr`指向的内存空间，释放的空间通常送入可用内存池，之后可以通过这三个函数重新分配。
9. `malloc`和`free`底层通常使用`sbrk`系统调用实现，这个系统调用扩充或者减小进程的堆，虽然`sbrk`可以扩充或者缩小进程的堆，但是一般`malloc`和`free`的实现不会减少进程的内存空间，释放的内存空间保存在`malloc`池中，而不是交给内核。
10. 大多数实现分配的空间要比请求的空间大一些，因为需要存储一些管理信息，如block的大小，指向下一个block的指针等等。因此，如果对超过一个分配区域的内存进行读写的话，会造成很严重的错误。
11. `free`一个已经释放了的块，`free`的不是`alloc`函数的返回值，没有进行`free`等等，都有可能造成很严重的后果。

## 环境变量
环境变量的形式是：
`name = value`
UNIX内核并不使用环境变量，通常都是应用程序使用这些环境变量。比如shell使用了大量的环境变量。

### 标准定义
ISO C定义了`getev`函数可以获取环境变量。但是ISO C没有定义任何环境变量。SUS环境变量包括POSIX.1和XSI环境变量。
除了获取环境变量，有时候我们也需要设置环境变量。ISO C没有定义获取环境变量的函数。SUS除了定义了ISO C，还定义了`putenv`, `setenv`和`unsetenv`对环境变量进行操作。

### `putenv`,`setenv`和`unsetenv`原型
``` c
#include <stdlib.h>

int putenv(char *string);
int setenv(const char *name, const char *value, int overwrite);
int unsetenv(const char *name);
```

### `putenv`,`setenv`和`unsetenv`性质
1. `putenv`，创建或者重置一个`name`。
2. `setenv`，如果`name`不存在，创建`name`；如果`name`存在，`rewrite`不为零，重写，`rewrite`为零，不进行重写。
3. `unsetenv`，移除`name`的定义。如果`name`不存在，不会出错。
4. 环境变量的修改和增加可能会遇到一些问题。因为环境变量存放在进程地址空间的最上面的一个不可扩展的空间。
**如果修改一个存在的`name`**：
当新的`value`的长度小于等于原来的`value`长度时，直接覆盖就行；
当新的`value`的长度大于原来的`value`长度时，只需要给新的`name-value`字符串分配空间就行了。使用`malloc`为新的字符串分配空间，然后将该字符串复制到新的空间，让environment list中`name`的指针指向新分配的存放字符串的空间即可。

**如果增加一个新的`name`，不仅需要给新的`name-value`分配空间，指针数组的元素也增加了，还需要给指针数组分配新的空间**。首先给新的`name-value`字符串分配空间，调用`malloc`先为字符串分配空间，然后将该字符串复制到这个空间：
如果这是第一次增加一个`name`，必须调用`malloc`为指针数组增加空间。将原来的指针数组复制到新分配的空间中，然后将新字符串的指针放在指针数组的尾部，然后存放一个空指针。最后让全局变量`char **environ`指向这个指针数组。如果原来的指针数组存放在栈顶之上，需要将它复制到堆中。需要注意的是，这个指针数组中的未修改的环境变量的指针还是指向栈顶的字符串上。
如果这不是第一个添加`name`，只需要使用`realloc`重新多分配一个指针数组的空间即可，然后将它指向新字符串的地址即可。


## `setjmp`和`longjmp`

### `setjmp`和`longjmp`原型
```c
#include <setjmp.h>

int setjmp(jmp_buf env);
void longjmp(jmp_buf env, int val);
```

### `setjmp`和`longjmp`属性
1. 自动变量存储在每个函数的栈帧中。
2. `setjmp`和`longjmp`实在栈上跳过若干调用栈，返回到当前函数调用路径上的某个函数中。

## `getrlimit`和`setrlimit`
每个进程能使用的资源都是有限的，可以使用`getrlimt`和`setrlimit`进行修改。它们都是XSI扩展，不是ISO C的定义。有些资源可以设置为`RLIM_INFINITY`，表示无限。

### `getrlimit`和`setrlimit`性质
1. 任何一个进程都可以将`rlim_cur`改为小于等于`rlim_max`。
2. 任何一个进程都可以将`rlim_max`改小，但是不能小于`rlim_cur`，且这个更改是不可逆的。
3. 只有root用户可以更改`rlim_max`。

它们的原型原型如下：
### `getrlimit`和`setrlimit`原型
``` c
#include <sys/time.h>
#include <sys/resource.h>

int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);
```

#### resource种类
- RLIMIT_AS
- RLIMIT_VMEM
- RLIMIT_DATA
- RLIMIT_SWAP
- RLIMIT_STACK
- RLIMIT_NPROC
- RLIMIT_FSIZE
- RLIMIT_NOFILE
- RLIMIT_NICE
- ...

#### 结构体
结构体`struct rlimt`的定义如下：```c
struct rlimit
{
    rlim_t rlim_cur;    // soft limit
    rlim_t rlim_max;    // hard limit
}
```

## 参考文献
1. 《APUE》第三版

