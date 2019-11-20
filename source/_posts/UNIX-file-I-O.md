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

### 创建文件
1. 创建一个只写文件（如果文件存在，将文件清零）：```c
creat(filename, mode);
open(filename, O_WRONLY|O_CREAT|O_TRUNC, mode);
```
2. 创建一个读写文件（如果文件存在，将文件清零）：``` c
open(filename, O_RDWR|O_CREAT|O_TRUNC);
```
3. 检测并创建一个只写文件（ 如果文件存在，错处，返回-1）：```
open(filename, O_WRONLY|O_CREAT|O_EXCL, mode);
```
4. 检测并创建一个读写文件（ 如果文件存在，错处，返回-1）：```
open(filename, O_RDWR|O_CREAT|O_EXCL, mode);
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
3. 如果`lseek`返回的当前文件偏移量不在文件结尾，`write`会覆盖掉相应位置的数据。

对于普通文件，写操作从文件的当前偏移量开始，如果打开文件时指定了`O_APPEND`选项，那么文件偏移量设置在文件结尾处。在一次写成功之后，文件偏移量增加实际写的字节数。

### 创建含有空洞的文件
如下所示代码，创建一个含有空洞的文件：```c
```

## I/O的效率
进程终止时，UNIX系统内核会关闭所有打开的文件描述符，但是并不会关闭标准输入和输出。
在选取`read`和`write`的buffer大小时，也有一定技巧。大多数文件系统都使用了预读(read ahead)技术。当进行顺序读取时，系统试图读入比应用所要求的更多数据，并且假设应用很快就会读这些数据。
在使用`ext4`文件系统时，它的磁盘块长度是4096，所以当BUFFER大于等于4096时，读写时间几乎不变。

## 文件共享

### I/O数据结构
UNIX支持在不同进程之间共享打开文件。这需要使用到内核用于I/O的数据结构。内核使用三种数据结构表示打开文件：进程表记录项，文件表项和节点表项。
1. 进程表记录项。每个进程在进程表中都有一个记录项，记录项中包含一张打开文件描述符表，每隔描述符占用一项，其中内容有：文件描述符标志和指向文件表项的指针。
2. 文件表项。内核为所有打开文件维持一张文件表。每个表项包含：文件状态标志，当前文件偏移量和指向该文件节点表项的指针
3. 节点表项。每个打开设备都有一个节点结构。包含文件的所有者，文件长度，指向文件实际数据块在磁盘上所在的指针等。

如果两个进程打开了同一个文件，每个进程都会获得各自相应文件的一个文件表项，这两个文件表项中的节点表项指针指向同一个节点表项。也有可能多个进程的文件描述符指向同一个文件表项。
自己的总结，每一个文件都有一个节点表项，记录文件长度和数据存储地址，而文件表项记录的是在节点表项的哪个位置进行什么操作，进程表记录项记录了每个进程打开了几个文件，每个文件的文件表项在哪里。

### `write`和`lseek`对当前文件偏移量的影响
1. `write`在写入完成后，在文件表项的当前文件偏移量上加上写入的字节数，如果当前文件偏移量超过了当前文件长度，更新节点表项中的文件长度，相当于文件长度增加了。
2. `lseek`只修改文件表项中的当前文件偏移量，不进行任何I/O操作。
3. `lseel`定位到文件尾端的时候，文件表项中当前文件偏移量被设置为节点表项中的当前文件长度。
4. 使用`O_APPEND`打开文件的时候，文件表项中的文件状态标志也会被修改，对于使用`O_APPEND`操作打开的文件，进行`write`操作相当于先将当前文件偏移量设置为节点表项中的文件长度，然后再`write`，即使先使用`lseek`将当前文件偏移量设置为`SEEK_SET`也不行，也是进行追加。所以在每次append之前不用先进行`lseek`，`lseek`了也白做。


## 原子操作
如果一个操作是原子操作，那么这个操作的所有步骤要么不执行，要不全部执行。

### 追加文件
指定`open`的`O_APPEND`选项实现追加操作，```c
fileno=open(filename, O_RDWR|O_APPEND);
write(file, buf, BUFSIZE);
```
追加文件是一个原子操作，如果不是原子操作的话，就相当于：```c
fileno = open(filename, O_RDWR);
lseek(fileno, 0, SEEK_END);
write(fileno, buf, BUFSIZE);
```
如果是单进程，上面两段代码是等价的，但是如果是多进程的话，下面代码就可能会出错。进程A lseek，进程B lseek，进程A write，进程B write。进程B的操作会覆盖进程A的操作。
所以这也就解释了使用选项`O_APPEND`后的操作，因为这个append的`write`是由两个系统调用组成的原子操作，先`lseek`，再普通的`write`。所以在调用`write`之前不用`lseek`，就算你`lseek`了也是白`lseek`。

### 读写原子操作
读写的原子操作原型如下：``` c
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t count, off_t offset);
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);
```
调用`pread`相当于调用`lseek`和`read`的原子操作，但是`pread`不改变当前文件偏移量。
调用`write`相当于调用`lseek`和`write`的原子操作，但是`pwrite`不改变当前文件偏移量。

### 创建文件原子操作
检查文件是否存在和创建文件是一个原子操作。如果这个操作不是原子操作，比如说是由`open`和`creat`两个函数调用组成的一个操作，它们不是一个原子操作。当前进程确定一个文件不存在，决定创建该文件。在`open`和`creat`调用之间，另一个进程创建了这个文件，并写入了数据。当前进程会再次创建这个文件，覆盖掉另一个进程写入的数据。

## `dup`和`dup2`复制文件描述符
UNIX系统提供了两个原子操作`dup`和`dup2`对一个指定的文件描述符进行复制。如果得到的新文件描述符和fd不同，那么这两个文件描述符共享同一个文件表项。
``` c
#include <unistd.h>

// dup返回的文件描述符一定是当前可用文件描述符中的最小值。和open一个文件类似。
int dup(int fd);

//可以通过fd2指定返回的新的文件描述符。
// 如果fd2和fd相等，返回fd2
// 如果fd2和fd不等，关闭fd2，然后返回fd2。
int dup2(inf fd, int fd2);
```

非原子操作的文件描述符复制可以通过`fcntl`实现。

## `sync`,`fsync`和`fdatasync`
UNIX系统在内核中设置了缓冲区高速缓存或者页高速缓存，大多数磁盘的I/O都通过缓冲区进行。当我们向文件写入数据时，内核通常先将数据复制到缓冲区中，然后排入队列，晚些写入磁盘，这方方式叫做**延迟写**。
等到内核需要使用缓冲区存放其他磁盘块数据时，它会把所有延迟写数据写入磁盘。为了保证磁盘上实际文件系统和缓冲区中内容的一致性，UNIX提供了三个函数，它们的原型如下：``` c
#include <unistd.h>

// 1.将所有修改过的块缓冲区排入队列，然后就返回，并不等待实际写磁盘操作结束。
// 通常情况下，update系统守护进程一般每隔30秒调用一次`sync`函数，这就保证了定期将内核块缓冲区的内容写入磁盘。命令sync(1)也会调用`sync`函数。
void sync(void);

// 2.只对文件描述符`fd`指定的一个文件起作用，等到写磁盘操作结束才返回。更新文件的数据和属性。
int fsync(int fd);

// 3.只对文件描述符`fd`指定的一个文件起作用，等到写磁盘操作结束才返回。只更新文件的数据。
int fdatasync(int fd);
```

## `fcntl`
fcntl是文件控制函数，它的原型如下：```c
#include <unistd.h>
#include <fcntl.h>

int fcntl(int fd, int cmd, ... /* arg */ );
```
fcntl有很多种功能，这一节先介绍以下五种：
1. 复制一个已有的描述符，设置`cmd`为`F_DUPFD`或者`F_DUPFD_CLOEXEC`。
2. 获取和设置文件描述符标志，设置`cmd`为`F_GETFD`或者`F_SETFD`。当前只有一个文件描述符标志，就是`FD_CLOEXEC`。
3. 获取和设置文件状态标志，设置`cmd`为`F_GETFL`或者`F_SETFL`。
获取文件状态标志时，介绍`open`时给出了许多文件状态标志。对于五个互斥的权限，需使用`O_ACCMODE`取得访问方式位，然后与相应的权限比对。对于其他的权限，将返回值和相应的标志进行与操作，判断是否设置了相应位。
设置文件状态标志位时，可以更改的几个权限有，`O_APPEND`,`O_NONBLOCK`,`O_SYNC`,`O_DSYNC`,`O_RSYNC`, `O_FSYNC`, `O_ASYNC`。
4. 获取和设置异步I/O所有权，设置`cmd`为`F_GETOWN`或者`F_SETOWN`。
5. 获取和设置记录锁，设置`cmd`为`F_GETLK`，或者`F_SETLK`或者`F_SETLKW`。

在修改文件描述符标志或者文件状态标志时，必须要先获得现在的标志值，然后对它进行修改，获得新的标志值，然后进行设置。不能单单设置一个标志值，否则会关闭以前设置的标志位。

## `ioctl`
这个有点看不懂。

## `/dev/fd`
UNIX提供了`/dev/fd`目录，其中包含了名为0, 1, 2的文件。打开`/dev/fd/0`,`/dev/fd/1`, `/dev/fd/2`相当于复制描述符n。即：```c
fd = open("/dev/df/0", mode);
fd = dup(0);
```
上述两行代码是相等的。文件描述符0和fd共享同一个文件表项。在Linux中，文件描述符被映射成指向底层物理文件的符号链接。比如打开`/dev/fd/0`时，实际上打开的是与标准输入关联的文件。返回的新文件描述符的mode和`/dev/fd`文件描述符的mode并不相关。所以，即使我们使用`O_RDWR` mode打开`/dev/fd/0`，也不能对`fd`进行写操作。
Linux下提供了`/dev/stdin`，`/dev/stdout`, `/dev/stderr`，它们和`/dev/fd/0`等都是一样的。在shell中，可以使用dev/fd作为参数，把标准输入和输出当做一个文件，可以像处理其他文件一样进行操作。



## 参考文献
1.《APUE》第三版
