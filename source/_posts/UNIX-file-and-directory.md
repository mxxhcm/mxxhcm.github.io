---
title: UNIX file and directory
date: 2019-11-21 10:18:13
tags:
 - UNIX
 - 文件
 - 目录
categories: UNIX
---

## 概述
这一节主要介绍文件系统的一些特征和文件的性质。相应的属性都存在`stat`函数给出的`struct stat`结构体中，每一个小节都会介绍结构体中的一个字段。

## `stat`, `fstat`, `fstatat`, `lstat`

### 函数原型
这一节主要介绍结构体`struct stat`的字段，以及如何获得某个目录的`struct stat`结构体。可以通过四个函数获得：``` c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);

#include <fcntl.h>           /* Definition of AT_* constants */
#include <sys/stat.h>

int fstatat(int dirfd, const char *pathname, struct stat *statbuf, int flags);
```

### 区别
1. `stat`将pathname指定的文件的有关信息存在放在`statbuf`中
2. `fstat`将文件描述符指定的文件的有关信息存在放在`statbuf`中
3. `lstat`和`stat`类似，但是当文件是一个符号链接时，`lstat`返回符号链接相关的信息，`stat`返回的是符号链接链向的文件。
4. `fstatat`相当于将前三个函数进行了整合：
当`dirfd`设置为`AT_FDCWD`时或者pathname是绝对路径时，设置`AT_SYSLINK_NOFOLLOW` flags时，相当于`lstat`。
当`dirfd`设置为`AT_FDCWD`时或者pathname是绝对路径时，`AT_SYSLINK_NOFOLLOW` flags不进行设置时，相当于`stat`。
当`dirfd`既不是`AT_FDCWD`而且pathname不是绝对路径时，就是`fstatat`处理的情况了。

### `struct stat`
这个结构体包括：
1. mode_t st_mode; 文件类型和全新啊
2. ino_t st_ino; i-node号
3. dev_t st_dev; 文件所在文件系统的设备号
4. dev_t st_rdev; 特殊文件所在文件系统的设备号
5. nlink_t st_nlink; 链接的数量
6. uid_t st_uid; 文件所有者的UID
7. gid_t st_gid; 文件所有者的GID
8. off_t st_size; 普通文件的字节数
9. struct timespec st_atime; 最后一次access的时间
10. struct timespec st_mtime; 最后一个modification的时间
11. struct timespec st_ctime; 最后一次file status change的时间
12. blksize_t st_blksize; 最合适的I/O block size
13. blkcnt_t st_blocks; 分配了多少个disk blocks


## 文件类型
UNIX系统的文件类型有以下几种：
1. 普通文件，可以是二进制文件，也可以是文本文件。除了二进制可执行文件必须遵循标准化格式外，其他的文件对于UNIX内核来说基本上没有区别。
2. 目录文件，包含了其他文件的名字，以及指向这些文件有关信息的指针。对一个目录具有读权限的任意进程都可以读目录的内容，但是只有内核才可以直接写目录文件。
3. block special file，提供对设备带缓冲的访问，每次访问以固定的长度进行。
4. character special file，
5. FIFO
6. socket
7. sysbolic link

## UID和GID

## 文件访问权限

## 新文件和新目录的权限

## `access`和`faccessat`

## 函数`umask`

## 函数`chmod`, `fchmod`和`fchmodat`

## sticky bit

### 文件访问权限小结
- S_ISUID，set user id,
- S_ISGID, set group id,
- S_ISVTX, stick bit,
- S_IRUSR, user read,
- S_IWUSR, user write,
- S_IXUSR, user exectu,
- S_IRGRP, group read,
- S_IWGRP, group write,
- S_IXGRP, group exectuble,
- S_IROTH, other read,
- S_IWOTH, other write,
- S_TXOTH, other exectuble,

可以对最后九项做一个简化版的表示：
S_IRWXU = S_IRUSR|R_IWUSR|S_IXUSR
S_IRWXG = S_IRGRP|S_IWGRP|S_IXGRP
S_IRWXO = S_IROTH|S_IWOTH|S_IXOTH

## 函数`chown`, `fchown`,`chownat`和`lchown`

## 文件长度

### 文件中的hole

## 文件截断

## 文件系统

## 函数`link`, `linkat`, `unlink`, `unlinkat`和`remove`

## 函数`rename`和`renameat`

## 符号连接

## 创建和读取符号链接

## 文件的时间

## 函数`futimens`, `utimensat`和`utimes`

## 函数`mkdir`，`mkdirat`和`rmdir`

## 读目录

## 函数`chdir`, `fchdir`和`getcwd`
每一个进程都有一个当前工作目录，**当前工作目录是进程的一个属性，**这个目录是搜索所有相对路径名的起点，不以"/"开始的路径名都是相对路径名。可以使用`chdir`后者`fchdir`改变当前进程的工作目录。它们的原型如下：```
#include <unistd.h>

int chdir(const char *path);
int fchdir(int fd);

char *getcwd(char *buf, size_t size);
```
`chdir`的参数是`path`，而`fchidir`的参数是文件描述符。

### `chdir`只会改变当前进程的work dir
**需要注意的一点是，`chdir`和`fchdir`只改变调用这个函数本身的进程，并不影响其他进程。**比如在shell中运行一个程序，在这个程序中更改了进程的当前工作目录，结束这个程序的执行时，shell的当前工作目录并不会改变，因为shell和我们刚才执行的程序属于两个不同的进程。因此，如果要改变shell进程自己的工作目录，应该使用shell直接调用`chdir`函数，所以`cd`命令内建在shell中。
`getcwd`是获得进程当前工作的绝对路径名。内核并不保存目录的完整路径名（linux除外），为了获得进程当前工作的绝对路径名。`getcwd`需要从当前工作目录开始，找到它的上一级目录，读取目录项，找到和工作目录i节点编号相同的目录项，得到对应的文件名。就这样一层一层的向上找，这就找到了绝对路径名。`getcwd`会follow符号链接，但是不会知道它是由哪里链接到这里的。

### `getcwd`的使用场景
当一个应用程序需要在经过一些列目录操作之后返回它刚开始的工作目录时。可以先使用`getcwd`获得最开始的工作目录，保存起来，最后再使用`chdir`进行恢复。
`fchdir`可以有更简单的操作，在刚开始时，保存目录的文件描述符。最后使用`fchdir`直接打开这个文件描述符。

## 设备特殊文件
这一小节主要介绍`st_dev`和`st_rdev`字段。
1. 每个文件系统所在的存储设备都由其主次设备号表示。设备号所用的数据类型是基本系统数据类型`dev_t`。主设备号标识设备驱动程序，次设备号标识特定的子设备。一个磁盘驱动器通常可以包含多个文件系统，同一个磁盘上的文件系统通常具有相同的主设备号，但是次设备号却不同。
2. 每个文件名中的`st_dev`是存放该文件名和其对应i节点的文件系统的设备号。
3. 可以使用两个宏`major`和`minor`访问主，次设备号，它们的参数都是`st_dev`。POSIX.1说明`dev_t`类型存在，但是没有定义它是什么，具体取值与实现相关。
4. 只有字符特殊文件和块特殊文件才有`st_rdev`值，同样使用`major`和`minor`两个宏访问相应的主次设备号。
5. 块特殊文件是包含随机访问文件系统的设备，比如硬盘驱动器，软盘驱动器和CD-ROM以及磁带等。


## 参考文献
1.《APUE》第三版
2.https://unix.stackexchange.com/questions/21251/execute-vs-read-bit-how-do-directory-permissions-in-linux-work
3.https://superuser.com/questions/168578/why-must-a-folder-be-executable/168583
