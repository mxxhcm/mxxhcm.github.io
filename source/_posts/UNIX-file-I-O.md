---
title: UNIX file I/O
date: 2019-11-18 19:19:20
tags:
 - UNIX
 - I/O
categories: UNIX
---

## 注意事项
1. `creat`以只写方式打开文件，不能进行读操作。
2. 为什么有了`open`要有`creat`，早期的`open`只支持0,1,2三个flag，不能打开不存在的文件，需要有单独的系统调用创建文件。而有了新的`open`以后就不需要`creat`了。
3. `open`返回的文件描述符一定是最小的未使用的文件描述符。

## 文件I/O
UNIX系统中的大多数文件I/O只用到了5个函数：`open`,`read`,`write`, `lseek`和`close`。不同的缓冲长度对`read`和`write`的影响。
本章介绍的函数通常被称为不带缓冲的I/O，不带缓冲的I/O指的是每个`read`和`write`都是内核中的一个系统调用，它们不是ISO C的组成部分，但是，它们都是POSIX.1和SUS的组成部分。
只要涉及在多个进程之间共享资源，原子操作的概念就非常重要。本章还进行一步讨论在多个进程之间如何共享文件，以及所涉及的内核有关数据结构。

## 文件描述符
对于内核而言，所有打开的文件都是通过文件描述符引用。文件描述符是一个非负整数。当打开或者创建一个新文件时，内核向进程返回一个文件描述符。当读，写一个文件时，使用`open`或者`creat`返回的文件描述符标识该文件，将其作为参数传递给`read`或者`write`。
UNIX系统shell把文件描述符0和进程的标准输入关联，文件描述符1和标准输出关联，文件描述符2和标准错误关联。为了提高系统的可读性，通常把它们换成符号常量`STDIN_FILENO`,`STDOUT_FILENO`和`STDERR_FILENO`，它们都在头文件`<unistd.h>`中定义。
文件描述符的变化范围是0到`OPEN_MAX-1`，早起的UNIX系统实现采用的上限值是19，现在的很多系统将它增加到63。（对于Linux, FreeBSD等的很多版本，文件描述符的变化范围几乎是无限的，只受到硬件资源的约束）

## `open`,`openat`和`creat`, `close`
函数原型：``` c
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>

       int open(const char *pathname, int flags);
       int open(const char *pathname, int flags, mode_t mode);

       int creat(const char *pathname, mode_t mode);

       int openat(int dirfd, const char *pathname, int flags);
       int openat(int dirfd, const char *pathname, int flags, mode_t mode);
```
参数`pathname`是路径名，可以是相对路径也可以是绝对路径，`flags`的选项有很多，它们定义在`<fcntl.h>`头文件中。flag参数必须在`O_RDONLY`,`O_WRONLY`和`O_RDWR`之中选且只能选一个。然后还有很多其他的可选flag，常见的有：`O_APPEND`，`O_CREAT`，`O_EXCL`, `O_DIRECTORY`等，使用`man 2 open`就可以查看。

### `openat` vs `open`
`open`和`openat`返回的一定是最小的没有使用的文件描述符。可以利用这一点可以在标准输入，标准输出，或者标准错误上打开新的文件。一个应用程序可以先关闭标准输出，然后打开另一个文件，执行打开操作前就能了解到该文件一定会在文件描述符1上打开。
`dirfd`参数是`open`和`openat`的区别，它们之间的关系有以下三种：
1. `pathname`指定的是绝对路径名，`dirfd`参数被忽略，`open`和`openat`一样。
2. `pathname`指定的是相对路径名，`dirfd`制定了相对路径名在文件系统中的开始地址，`dirfd`参数通过打开相对路径名的目录来获取。
3. `pathname`指定了相对路径名，`dirfd`的参数是特殊值`AT_FDCWD`，这种情况下，路径名是在当前工作目录中获取，`openat`和`open`在操作上类似。

### `openat`作用
为什么增加`openat`函数，它的目的是解决两个问题：
1. 让线程可以使用相对路径名打开目录中的文件，而不再只能打开当前工作目录。同一进程中的所有线程共享相同的当前工作目录。
2. 避免time-of-check-to-time-of-use错误。它的基本思想是，如果有两个基于文件的系统调用，第二个调用的结果依赖于第一个调用的结果，那么程序是脆弱的。因为两个调用并不是原子操作，在两个函数调用之间，文件可能改变了，这就造成了第一个调用的结果不再有效，使得程序的最终结果是错误的。

### 文件名和路径名过长
当文件名和路径名过长时，是截断为系统允许的最长量还是返回出错信息？这个是由系统的历史形成的。通常BSD和Linux总是会返回出错，而System V和Solaris等不一定。
具体的可以根据POSIX.1定义的常量`_POSIX_NO_TRUC`决定是截断还是出错。根据文件系统的类型，这个值可以变换。可以使用`fpathconf`或者`pathconf`查询目录具体支持哪种行为。
如果`_POSIX_NO_TRUC`有效，当路径名超过`PATH_MAX`或者路径名中的任一文件名超过`NAME_MAX`时，返回出错，并将`errno`设置为`ENAMETOOLONG`。

### `create`和`open`
`create`其实相当于指定了`open`的flags为`O_WRONLY|O_CREAT|O_TRUNC`。
为什么有了`open`还要有`creat`，在早期的UNIX版本中，`flags`只能为0,1或者2。无法打开一个不存在的文件。因此需要另一个系统调用`creat`创建新文件。现在的`open`系统调用提供了`O_CREAT`和`O_TRUNC`选项，也就不需要`creat`了。

`creat`的不足：`creat`以**只写方式**打开所创建的文件，即创建新文件之后，只能对新文件进行写操作，不能进行读操作。如果要创建一个临时文件，先写文件，然后再读文件。在`open`的老版本时，即不能打开不存在的文件时，需要先使用`creat`创建新文件，然后关闭该文件，然后使用`open`读文件。现在的话，可以使用以下方式实现创建新文件并进行读写：``` c
open(path, O_RDWR|O_CREAT|O_TRUNC, mode);
```

### `close`
调用`close`关闭一个已经打开的文件。函数原型：```c
#include <unistd.h>

int close(int fd);
```

## `lseek`, `read`, `write`
它们的原型如下所示：
``` c
#include <sys/types.h>
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```
### 当前文件偏移量
1. 每一个打开文件都有一个和它相关联的当前文件偏移量(current file offset)。它通常是一个非负整数，用来度量从文件开始处的字节数。
2. 通常情况下，读写操作都是从current file offset开始的，并且使偏移量增加读写的字节数。
3. 除了指定`O_APPEND`选项外，打开一个文件时，默认的current file offset都是0。
4. `whence`有三个取值，`SEEK_SET`, `SEEK_CUR`, `SEEK_END`。`lseek`成功执行，返回的offset等于`whence+offset`，对于`SEEK_CUR`和`SEEK_END`来说，参数`offset`可正可负，只要保证返回的current file offset非负即可。
5. `lseek`中的`l`表示`long`。
6. current file offset可以大于文件长度，这种情况会在文件中构成一个空洞。空洞不要求占据磁盘上的存储区。

### `read`
``` c
ssize_t read(int fd, void *buf, size_t count);
```
`read`函数从文件描述符标示的文件中读取至多`count`个字节到`buf`指定的位置。如果操作成功的话，返回读到的字节数，如果已经到了文件结尾，返回0。出错的话，返回-1，设置errno。
在以下几种情况下，读到的字节数可能少于`count`：
1. 读普通文件时，在读满`count`个之前就已经到了文件尾端。
2. 某个信号造成中断时，而已经读取了部分数据时。
3. 从终端设备读时，通常一次最多读一行。
4. 从网络读时，网络中的缓冲机制。
5. 从管道或者FIFO读取时，管道包含的字节数少于`count`。
6. 从面向记录的设备读时，一次最多返回一个记录。
7. 第二个参数`void*`表示通用指针。
8. 返回值`ssize_t`是有符号类型，因为它需要返回正整数字节，0和-1。
9. 第三个参数`size_t`是一个无符号类型。

### `write`
``` c
ssize_t write(int fd, const void *buf, size_t count);
```
它的返回值通常和`count`一样，否则就是出错了。出错的常量原因有：
1. 磁盘满了，
2. 超过了一个给定进程的文件长度限制。

对于普通文件，写操作从文件的当前偏移量开始，如果打开文件时指定了`O_APPEND`选项，那么文件偏移量设置在文件结尾处。在一次写成功之后，文件偏移量增加实际写的字节数。

### 创建含有空洞的文件
如下所示代码，创建一个含有空洞的文件：```c

```

## I/O的效率

## 文件共享

## 原子操作

## `dup`和`dup2`

## `sync`,`fsync`和`fdatasync`

## `fcntl`

## `ioctl`

## `/dev/fd`

## 参考文献
1.《APUE》第三版
