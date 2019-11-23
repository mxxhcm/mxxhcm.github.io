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
这一节主要介绍文件系统和文件的一些属性，这些属性都存在`stat`函数给出的`struct stat`结构体中，`struct stat`的部分字段如下所示，接下来的小节会对这些字段进行详细的介绍：

### `struct stat`
这个结构体包括：
1. `mode_t st_mode;` 文件类型和文件权限控制
2. `ino_t st_ino;` 文件存放的i-node号
3. `dev_t st_dev;` 文件所在文件系统的设备号
4. `dev_t st_rdev;` 特殊文件所在文件系统的设备号
5. `nlink_t st_nlink;` 链接到文件指向的i-node节点的数量，即hardlink
6. `uid_t st_uid;` 文件所有者的UID
7. `gid_t st_gid;` 文件所有者的GID
8. `off_t st_size;` 普通文件的字节数
9. `struct timespec st_atime;` 最后一次access文件内容的时间
10. `struct timespec st_mtime;` 最后一个modification文件内容的时间
11. `struct timespec st_ctime;` 最后一次文件i-node 内容change的时间
12. `blksize_t st_blksize;` 最合适的I/O block size
13. `blkcnt_t st_blocks;` 分配了多少个disk blocks

## 文件系统
UNIX文件系统有多种实现，比如基于BSD的UNIX文件系统(UFS)，linux的ext4文件系统，以及Mac OS X的HFS文件系统。本节拿UFS举例子。
一个磁盘可以有一个或者多个partition，每个partition可以包含文件系统也可以不包含。对于UFS文件系统，它由boot blocks，super block，cylinder groups组成。每一个cylinder group由super block copy, config info, i-node图，block bitmap以及i-nodes和data blocks组成。i-node节点是固定长度的记录项，它包含有关文件的大部分信息。
1. i-node节点包含了文件的：文件类型，文件访问权限，文件长度，和指向文件数据的指针等，`stat`绝大部分的数据都取自i-node。
2. data blocks中存放实际的数据，包含data blocks和directory blocks。data blocks存放文件的实际数据。目录也是文件，directory blocks也是data block，只不过它是存放目录文件中所有directory entry(目录项)的data blcok，一个directory entroy包括文件名和i-node编号。i-node节点编号的数据类型是`ino_t`。
3. hard links：在每一个i-node中，都有一个链接计数，记录指向它的目录项个数，POSIX.1常量`LINK_MAX`指定了一个文件连接数的最大值。链接计数可以从`st_nlink`中得到，类型是`nlink_t`。hard link创建一个新的目录项指向链接目标的i-node节点，删除原来的目录项对新的目录项没有影响。只有当指向i-node节点的链接数等于0时且没有进程使用该文件时，可以删除该文件。
4. symbolic links：symbolic links文件的实际内容，也就是存储在data blocks中的内容，是该symbolic link指向的文件的名字。symbolic link和hard link不一样，它有自己的i-node号，删除它指向的文件，这个symbolic link会失效，变成一个dead link。它的文件类型是`S_IFLNK`。
5. 一个目录项不能指向另一个文件系统的i-node，所以hard link不能跨越文件系统。
6. 在不更换文件系统的情况下为一个文件重命名的话。文件的实际内容没有变，只是构造了一个新的目录项，它的i-node就是待重命名的文件，然后删除老的目录项。链接的计数并不会改变，这就是`mv`命令的工作方式。
7. i-node节点指向的数据块也可以存放目录项，在i-node的`st_mode`中可以确定它的类型是`S_IFDIR`。创建一个名为`testdir`的空目录，它的链接计数是2，一个是`.`目录，一个是`testdir`。`testdir`的上一级目录，它的链接计数是3（假设上一级目录只包含`testdir`），一个是这个目录的名字，一个是`.`，一个是`testdir`中的`..`。

关于linux文件系统，可以查看[linux file system](https://mxxhcm.github.io/2019/05/07/linux-file-system/).

### 设备特殊文件
这一小节主要介绍`st_dev`和`st_rdev`字段。
1. 每个文件系统所在的存储设备都由其主次设备号表示。设备号所用的数据类型是基本系统数据类型`dev_t`。主设备号标识设备驱动程序，次设备号标识特定的子设备。一个磁盘驱动器通常可以包含多个文件系统，同一个磁盘上的文件系统通常具有相同的主设备号，但是次设备号却不同。
2. 每个文件名中的`st_dev`是存放该文件名和其对应i节点的文件系统的设备号。
3. 可以使用两个宏`major`和`minor`访问主，次设备号，它们的参数都是`st_dev`。POSIX.1说明`dev_t`类型存在，但是没有定义它是什么，具体取值与实现相关。
4. 只有字符特殊文件和块特殊文件才有`st_rdev`值，同样使用`major`和`minor`两个宏访问相应的主次设备号。
5. 块特殊文件是包含随机访问文件系统的设备，比如硬盘驱动器，软盘驱动器和CD-ROM以及磁带等。

## 文件类型
文件类型信息包含在`struct stat`的`st_mode`字段中。UNIX系统的文件类型有以下几种：
1. 普通文件，可以是二进制文件，也可以是文本文件。除了二进制可执行文件必须遵循标准化格式外，其他的文件对于UNIX内核来说基本上没有区别。使用`S_ISREG`宏进行判断。
2. 目录文件，包含了其他文件的名字，以及指向这些文件有关信息的指针。对一个目录具有读权限的任意进程都可以读目录的内容，但是只有内核才可以直接写目录文件。使用`S_ISDIR`宏进行判断。
3. block special file，提供对设备带缓冲的访问，每次访问以固定的长度进行。使用`S_ISBLK`宏进行判断。
4. character special file，提供对设备不带缓冲的访问，每次访问长度可变。系统中的设备要么是block special file要是character special file。使用`S_ISCHR`宏进行判断。
5. FIFO，用于进程间通信。使用`S_ISFIFO`宏进行判断。
6. socket，用于进程间的网络通信。使用`S_ISSOCK`宏进行判断。
7. symbolic link，指向另一个文件。使用`S_ISLNK`宏进行判断。

## `stat`, `fstat`, `fstatat`, `lstat`
可以通过`stat`等函数获得文件（各类文件）的`struct stat`结构体，它们的原型如下：

### 函数原型
``` c
#include <sys/stat.h>

int stat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);
int fstatat(int dirfd, const char *pathname, struct stat *statbuf, int flags);
```

### 性质
1. `stat`将pathname指定的文件的有关信息存在放在`statbuf`中
2. `fstat`将文件描述符指定的文件的有关信息存在放在`statbuf`中
3. `lstat`和`stat`类似，但是当文件是一个symbolic link时，`lstat`返回symbolic link相关的信息，`stat`返回的是symbolic link链向的文件。
4. `fstatat`相当于将前三个函数进行了整合：
当`dirfd`设置为`AT_FDCWD`时或者pathname是绝对路径时，设置`AT_SYSLINK_NOFOLLOW` flags时，相当于`lstat`。
当`dirfd`设置为`AT_FDCWD`时或者pathname是绝对路径时，`AT_SYSLINK_NOFOLLOW` flags不进行设置时，相当于`stat`。
当`dirfd`既不是`AT_FDCWD`而且pathname不是绝对路径时，就是`fstatat`处理的情况了。

## hard link和函数`link`, `linkat`, `unlink`, `unlinkat`和`remove`
任何一个文件可以有多个目录项指向它的i-node节点。可以使用`link`和`linkat`创建hard link，使用`unlink`和`unlinkat`, `remove`删除hard link。

### hard link的特点
1. 使用hard link链接的文件具有相同的inode和data block。
2. 删除一个hard link不会影响其他具有相同inode号的文件。
3. 只能对已经存在的文件创建hard link。
4. 不能跨越文件系统创建hard link，因为每个文件系统的i-node节点都是单独的。
5. 不能对目录创建hard link，可能会形成循环。

### `link`和`linkat`的函数原型
``` c
#include <unistd.h>

int link(const char *oldpath, const char *newpath);
int linkat(int olddirfd, const char *oldpath, int newdirfd, const char *newpath, int flags);
```
### `link`和`linkat`的函数性质
1. `link`和`linkat`应该是原子操作。
2. `linkat`和`link`类似。

### `unlink`和`unlinkat`的函数原型
``` c
#include <unistd.h>

int unlink(const char *pathname);
int unlinkat(int dirfd, const char *pathname, int flags);
```

### `unlink`和`unlinkat`的函数性质
1. 如果`pathname`是hard link，这两个函数删除`pathname`的相应目录项，将`pathname`指向文件的i-node的链接数减一。
2. 如果`pathname`是symbolic link，`unlink`删除的是symbolic link本身，而不follow它。
3. 给出symbolic link时，没有任何函数能够删除symbolic link指向的文件。
4. 如果链接数等于0，并且内核检查打开该文件的进程个数，如果它们都等于0。`unlink`会删除这个文件。如果链接数或者打开文件的进程个数大于0，不会对文件做修改。
5. `unlinkat`和`unlink`类似。

### `remove`的函数原型
``` c
#include <stdio.h>

int remove(const char *pathname);
```

### `remove`的函数性质
1. `remove`可以解除对文件或者目录的hard link。
2. 作用于目录时，`remove`相当于`rmdir`。
3. 作用于文件时，`remove`相当于`unlink`。

## symbolic link和函数`symlink`,`symbolinkat`, `readlink`, `readlinkat`
hard links有一些限制：
1. 链接和文件必须处于同一文件系统中，
2. 只有root用户才能创建指向目录的链接。

### symbolic link的特点
而symbolic link可以避开hard link的这些限制。
1. symbolic可以看成文件的间接指针，它有自己的文件属性和权限，有自己的i-node号和data block，而所有指向一个i-node节点的hard link拥有相同的i-node。
2. symbolic link对于它指向的对象没有任何文件系统的限制，任何用户都可以创建指向目录的symbolic link。
3. 使用文件名字为参数调用文件函数时，需要知道这个函数是否处理symbolic link的。如果函数具有处理symbolic link的功能，那么它处理的是symbolic link指向的文件，否则它直接处理这个symbolic link本身。
如下所示，是常见的函数是否处理symbolic link，`fstat`和`fchmod`因为处理的是文件描述符，这个文件描述符是否是symbolic link本身通常是由`open`决定的。
`access`,`chdir`, `chmod`, `chown`，`creat`, `exec`, `link`, `open`, `opendir`, `pathconf`, `stat`, `truncate`跟随symbolic link。
`lchown`,`lstat`, `readlink`, `remove`, `rename`, `unlink`不跟随symbolic link。
4. 使用symbolic link可能在文件系统中引入循环。`unlink`不跟随symbolic link，所以使用`unlink`消除symbolic link。但是hard link的循环很难消除。这也是为什么`link`函数不允许构造指向目录的hard link的原因。
5. symbolic link可以指向系统中并不存在的文件。所以`creat`的参数可以是symbolic link。

创建和读取symbolic link的函数原型如下：
### `symlink`和`symlinkat`函数原型
``` c
#include <unistd.h>

int symlink(const char *target, const char *linkpath);
int symlinkat(const char *target, int newdirfd, const char *linkpath);
```

### `symlink`和`symlinkat`函数性质
1. `linkpath`和`target`不需要在同一个文件系统，而且`target`甚至可以不存在
2. `symlinkat`和`symlink`类似。

### `readlink`和`readlinkat`函数原型
``` c
#include <unistd.h>

ssize_t readlink(const char *pathname, char *buf, size_t bufsiz);
ssize_t readlinkat(int dirfd, const char *pathname, char *buf, size_t bufsiz);
```

### `readlink`和`readlinkat`函数性质
1. `open`函数跟随symbolic link，而`readlink`打开symbolic link本身，并且读取symbolic link中的名字
2. `readlink`组合了`open`,`close`和`read`的所有操作。如果函数成功执行，返回读入buf的字节数。
3. `readlinkat`和`readlink`类似。



## 和进程相关的UID和GID
每一个进程有6个或者更多和它相关的ID：
### 实际UID和GID
real UID和real GID，用来表示当前用户。

### 有效UID和GID
effective UID和effective UID，决定我们的文件访问权限。通常情况下，effective UID以及effective GID和real UID以及real GID一样。

### Set-User-ID和Set-Group-ID
saved set-user-ID和saved set-group-ID，在执行一个程序时，包含了有效user ID和有效group ID的副本。
每一个文件都有一个UID和GID，它们的值在`st_uid`和`st_gid`中。
当执行一个程序文件时，进程的effective UID通常就是real UID，而effective GID通常就是real GID。但是可以在`st_mode`中设置一个特殊的flag，意思是当执行此文件时，将执行此文件的进程的effective UID设置为文件所有者的UID。同样，还有另一个特殊的flag，它将执行此文件的进程的effective GID设置为文件组所有者的UID。这两个标志位被记为set-user-ID bit和set-group-ID bit，它们都存放在`st_mode`中，可以使用`S_ISUID`和`S_ISGID`测试。
**运行set UID程序的进程通常会获得额外的权限！！！所以要格外注意。**

## 文件和目录的访问权限
`st_mode`中还包含了文件的访问权限。对于所有文件类型（不单单是文件和目录），都有三种访问权限：

### r-读权限
读权限查询目录名内的数据。

### w-写权限
- 新建文件与目录 
- 删除文件或者目录 
- 重命名以及转移文件或者目录
	
### x-可执行权限
- 进入某目录 
- 切换到该目录（cd命令）
- 对于只具有可执行权限的目录，可以使用`cd`进入该目录，也可以打开该目录的文件，或者进入下一级目录。但是需要我们知道它们的名字，不能使用`ls`命令查看，因为没有读目录的权限。。

!!!能不能进入某一目录只与该目录的x权限有关，如果不拥有某目录的x权限，即使拥有r权限，那么也无法执行该目录下的任何命令。
但是即使拥有了x权限，但是没有r权限，能进入该目录但是不能打开该目录，因为没有读取的权限，但是可以进入下一级目录或者打开当前目录的文件（因为不能读目录，所以需要知道下一级目录的名字或者当前目录下要打开的文件的名字），路径上的所有目录都必须有可执行权限。。

### sticky bit
`S_ISVTX`较老版本的UNIX叫做表示stick bit，而新版UNIX叫做saved-text bit，这也是`S_ISVTX`名字的由来。这一位的作用是，当一个可执行文件的这一位被设置了，当该程序第一次被执行，在它终止时，程序正文部分的一个副本，即机器指令，仍然保存在交换区中，这使得下次执行该程序时能较快的载入内存。因为通常的UNIX文件系统中，文件的数据块都是随机存放的，相对来说，交换区被当做一个连续文件来处理。而现在的UNIX系统都使用了虚拟存储系统和快速文件系统，已经不需要这种技术了。
现在系统扩展了stick bit的使用，SUS允许对目录设置目录的stick bit，如果一个目录设置了stick bit，只有对该目录具有写权限的用户并且满足下列条件之一，才能删除或者重命名该目录下的文件：
1. 拥有该文件
2. 拥有此目录
3. 是root用户

比如/tmp目录，设置了stick bit，任何用户都可以在这个目录下创建文件。任意用户,组和其它对这两个目录的权限都是读写和执行。但是用户不能删除和重命名属于其他人的文件。

### 十二个访问权限位
将`rwx`和user, group以及other进行组合，总共有九个访问权限位，再加上`S_ISUID`,`S_ISGID`和`S_ISVTX`三个特殊位：
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

可以对最后九项做一个简洁版的表示：
S_IRWXU = S_IRUSR|R_IWUSR|S_IXUSR
S_IRWXG = S_IRGRP|S_IWGRP|S_IXGRP
S_IRWXO = S_IROTH|S_IWOTH|S_IXOTH


### 文件和目录的操作规则
1. 使用名字打开任意类型的文件时，对于文件名字中包含的每一个目录，包括当前工作目录，都应该具有执行权限。
2. 对于一个文件的读权限决定了我们能够打开先有文件进行读操作。这与`open`函数的`O_RDONLY`和`O_RDWR`有关。
3. 对于一个文件的写权限决定了我们能够打开先有文件进行写操作。这与`open`函数的`O_WRONLY`和`O_RDWR`有关。
4. 如果要在`open`函数中指定`O_TRUNC`标志，必须对该文件拥有写权限。
5. 为了在一个目录中创建新文件，必须对这个目录具有写权限和执行权限。
6. 为了删除一个现有文件，必须对包含该文件的目录拥有写权限和执行权限，而不必对文件本身拥有读权限和写权限。
7. 如果使用7个`exec`函数中的任何一个执行某个文件，都必须拥有该文件的写权限。

### 访问权限检测
进程每次打开，创建或者删除一个文件时，内核就会进行文件访问权限测试，这种测试可能涉及到文件的UID，文件的组GID，进程的effective UID和effective GID。文件的UID和文件的GID都是文件的属性，而effective UID和effective GID是进程的属性。
内核进行访问权限测试的步骤如下：
1. 如果进程的effective UID是0，结束权限判断，允许各项访问。否则跳转第2步进行判断。
2. 如果进程的effectiev UID等于文件UID，也就是`st_uid`，结束权限判断，根据访问权限位允许相应操作。否则跳转第3步。
3. 如果进程的effective GID等于文件的GID，结束权限判断，根据访问权限的设置允许相应的操作，否则跳转第4步。
4. 如果不满足前三条，就按照若其他用户的访问权限位判定操作是否合法。

总结一下，就是依次判断effective UID是不是等于root，effective UID是不是等于`st_uid`，或者effective GID是不是等于`st_gid`，如果都不满足，就按照其它权限判定当前进程对文件的操作是否被允许。按照顺序来判断，满足一个就不用判断后面的了。

## 函数`umask`
前面介绍了和文件相关的9个访问权限位，使用进程创建文件时，文件的权限可以由`umask`修改，相当于在原有的mode上减去`umask`指定的mode。函数`umask`的原型如下：

### 函数原型
``` c
#include <sys/stat.h>

mode_t umask(mode_t cmask);
```

### 特点
1. 这个函数的作用是去掉`cmask`中指定的权限，返回之前的mode。
2. **在程序中使用`open`和`creat`等创建新文件时，如果想要确保指定的访问限权被激活，必须在进程运行时修改`umask`的值。否则，`umask`可能会覆盖掉我们创建文件时指定的mode。**
3. shell中有内置的`umask`命令，SUS要求shell的`umask`除了支持八进制的拒绝权限外，还要支持符号格式的指定许可的权限。使用``` shell
umask -S
```
查看。

## 新文件和目录的UID和GID
创建文件时，新文件的UID被设置为进程的effective UID。关于新文件的GID，可以选择以下两种方式中的一个进行设置：
1. 新文件的GID可以是进程的effective GID
2. 新文件的GID可以是它所在目录的GID。

不同的UNIX实现有不同的设置，这里拿linux来说，Linux 3.2.0以后，新文件的GID取决于它所在目录的set-group ID bit是否被设置，如果被设置了，新文件的ID就是它所在目录的GID，否则就是进程的effective GID。
新目录的UID和GID和文件一样。

## 函数`mkdir`，`mkdirat`和`rmdir`
可以使用`mkdir和`mkdirat`创建目录，使用`rmdir`删除目录。它们的原型如下：

### 函数原型
``` c
#include <sys/stat.h>

int mkdir(const char *pathname, mode_t mode);
int mkdirat(int dirfd, const char *pathname, mode_t mode);
int rmdir(const char *pathname);
```

### 性质
1. `mkdir`创建一个空目录，自动包含`.`和`..`项。文件的访问
2. 目录的权限默认权限是进程的`umask`给出的。对于目录，我们至少需要指定execute位。
3. 目录的UID和GID跟进程`create`文件时一样。
4. `mkdirat`和`mkdir`类似。
5. `rmdir`删除目录。

## 函数`access`和`faccessat`
使用`open`函数打开文件时，内核使用进程的effective UID和effective GID检测它对文件的访问权限。
`acess`使用进程的real UID和real GID进行权限访问测试。访问权限测试步骤和之前介绍的四步一样，只不过使用real UID和real GID代替了effective UID和effective GID。

### 函数原型
`access`和`faccessat`的原型如下：``` c
#include <unistd.h>

int access(const char *pathname, int mode);
int faccessat(int dirfd, const char *pathname, int mode, int flags);
```

### 参数和区别
1. `mode`可选参数有，`F_OK`，`R_OK`, `W_OK`,`X_OK`，其中`F_OK`表示测试这个文件是否存在。
2. `faccessat`和`access`在两种情况下相同，`dirfd`设置为`AT_FDCWD`且`pathname`四相对路径和`pathname`是绝对路径。否则的话，`faccessat`就是测试相对于`dirfd`指向的打开目录下的`pathname`的权限。
3. 如果`flags`设置为`AT_EACCESS`的话，权限访问检测使用的是effective UID和effective GID而不是real UID和real GID。

## 函数`chmod`, `fchmod`和`fchmodat`
文件的访问权限可以使用`chmod`, `fchmod`和`fchmodat`进行修改，它们的原型如下：
### 原型
```c
#include <sys/stat.h>

int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int dirfd, const char *pathname, mode_t mode, int flags);
```

### 特点
1. `chmod`在指定的文件上进行操作
2. `fchmod`是对已经打开的文件文件描述符进行操作。
3. `fchmodat`和`chmod`在两种情况下是相等的，当`pathname`是绝对路径时，以及`dirfd`设置为`AT_FDCWD`且`pathname`是相对路径的时候。否则，`fchmodat`操作相对于打开目录的pathname。
4. 当flags设置了`AT_SYMLINK_NOFLOLLW`时，不会followsymbolic link。
5. 在以下两种情况下，`chmod`函数自动清除两个权限位：
    - 新创建文件的GID可能不是调用进程的effective GID。新文件的GID可能是父目录的GID。如果新文件的GID不等于进程的effective GID，而且进程没有root权限，set-group-id位会被自动关闭。
    - stick bit的设置


## 函数`chown`, `fchown`,`chownat`和`lchown`
可以使用`chown`, `fchown`,`chownat`和`lchown`更改文件的UID和GID。它们的原型如下：
### 函数原型
``` c
#include <unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int lchown(const char *pathname, uid_t owner, gid_t group);
int fchownat(int dirfd, const char *pathname, uid_t owner, gid_t group, int flags);
```

### 特点
1. 当文件不是symbolic link时，所有的函数操作类似。
2. 当文件是symbolic link时，`lchown`和设置了`AT_SYMLINK_NOFOLLOW`标志的`fchownat`都修改的是symbolic link本身，而不是它链接的对象。
3. `fchown`操作的是`fd`指向的已经打开的文件，因为它在一个已经打开的文件上操作，所以它不能修改symbolic link本身。
4. `fchownat`和`fchown`或者`lchown`在下面两种情况下是等价的：当pathname是决定路径或者`dirfd`设置为`AT_FDCWD`时且pathname是相对路径时。在这两种情况下，如果`flags`设置了`AT_SYMLINK_NOFOLLOW`标志，`fchownat`和`lchown`一样，否则`fchownat`和`fchown`一样。如果不是这两种情况，`fchowat`是操作相对于打开目录的pathname。
5. 根据`_POSIX_CHOWN_RESTRICTED`常量的值，可以查询是否只有超级用户才能更改文件的所有者，如果这个常量对指定的文件有效，那么
    - 只有root进程才能更新该文件的UID，普通用户不能修改其他用户文件的UID
    - 如果进程拥有此文件(进程的effective UID等于文件的UID)， 可以更改这些文件的GID，但是只能更改到进程的effective GID或者继承的附属组ID之一。
6. 如果这些函数由非root进程调用，在成功返回时，set UID和set GID位都会被清楚。


## 函数`rename`和`renameat`
可以使用函数`rename`和`rename`对文件或者目录进行重命名。它们的原型如下：
### 函数原型
``` c
#include <stdio.h>

int rename(const char *oldpath, const char *newpath);
int renameat(int olddirfd, const char *oldpath, int newdirfd, const char *newpath);
```

### 性质
1. ISO C对文件定义了`reanme`，但是ISO C没有对目录定义该函数。POSIX.1扩展了定义，使得它`rename`可以处理目录和symbolic link。
2. 如果`oldpath`指向普通文件，那么为文件进行重命名。如果`newname`存在，它不能是一个目录。如果它不是目录，先将该目录项删除，然后将`oldname`重命名为`newname`。进程需要对`oldpath`和`newpath`的父目录都拥有写和执行权限，因为`rename`会修改这两个目录。
3. 如果`oldpath`指向目录，为目录重命名。如果`newname`已经存在，那么它必须是一个目录，而且应该是空目录。如果`newname`目录存在且为空，先将它删除，然后将`oldname`重命名为`newname`。且`newname`不能包含`oldname`，比如不能将`/usr/foo`重命名为`/usr/foo/testdir`，因为不能删除`/usr/foo`目录。
4. 如果`oldpath`指向symbolic link，`rename`不follow symbolic link，直接处理symbolic link本身。
5. 不能对`.`和`..`进行重命名。或者说`.`和`..`不能出现在`oldpath`和`newpath`上。
6. 当`oldpath`和`newpath`指向同一个文件，函数不做任何操作直接返回。
7. `renameat`和`rename`类似。

## 文件长度
`struct stat`中的`st_size`以字节为单位表示文件的长度。这个字段只对普通文件，目录文件和symbolic link有意义。
对于普通文件，它的长度可以是0，在开始读这种文件时，得到EOF标志。
对于目录，文件长度通常是一个数的整数倍。
对于symbolic link，文件长度是链接指向的文件名的实际字节数，不包含null字节。
对于现在的大多数UNIX系统，提供了`st_blksize`和`st_blocks`字段。`st_blksize`表示对文件I/O适合的块长度，第二个是所分配的固定大小的block数量。

### 文件中的hole
1. 普通文件可以有hole，hole是当前文件偏移量超过文件尾端，然后进行写入造成的。
2. 对于同样长度的有空洞和没有空洞的文件来说，它们所占用的blocks块数是不同的。这里说的同样长度指的是字节数，使用`ls -l`列出来的长度，那些空洞不占用磁盘上的存储区。
3. 使用`cat`进行复制时，会将空洞使用0字节填满。
4. 使用`du -s file.txt`可以查看文件所占用的blocks数量，这些blocks中还有一些用来存放指向实际数据块的指针。

### 文件截断
可以使用`truncate`和`ftruncate`截断文件，函数原型是：
#### 函数原型
``` c
#include <unistd.h>
#include <sys/types.h>

int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

#### 特点
1. 当`length`长度小于原来文件的长度时，超过`length`的进行截断。
2. 当`length`长度大于原来文件的长度时，原来的文件长度到`length`之间的数据被读作0。
3. `ftruncate`操作的是文件描述符，`truncate`操作的是文件。

## 文件的时间
### 文件的三个时间
每一个文件守护三个时间：
1. `st_atim`，记录文件数据的最后访问时间。使用`ls -u`查看。
2. `st_mtim`，记录文件数据的最后修改时间。默认的`ls`显示的就是文件的最后修改时间。
3. `st_ctim`，记录i-node状态的最后更改时间。使用`ls -c`查看。常见的许多操作都会影响i-node，主要就是`i-node`中存放的那些信息，更改文件权限，更改文件的`st_uid`和`st_gid`，文件的链接数等等。

这个让我想不明白的是为什么`link`,`unlink`,`creat`等会影响所使用文件的父目录的inode节点。

### 函数`futimens`, `utimensat`和`utimes`原型
``` c
#include <sys/stat.h>

int utimensat(int dirfd, const char *pathname, const struct timespec times[2], int flags);
int futimens(int fd, const struct timespec times[2]);

#include <sys/time.h>

int utimes(const char *filename, const struct timeval times[2]);
```

### 函数`futimens`, `utimensat`和`utimes`性质
1. 如果`times`为空，访问时间和修改时间都设为当前时间
2. 如果`times`指向两个`timespec`结构的数组，任一元素的`tv_nsec`字段的值为`UTIME_NOW`，相应的时间戳设置为当前时间，忽略相应的`tv_sec`字段。
3. 如果`times`指向两个`timespec`结构的数组，任一元素的`tv_nsec`字段的值为`UTIME_OMIT`，相应的时间戳保持不变，忽略相应的`tv_sec`字段。
4. 如果`times`指向两个`timespec`结构的数组，`tv_nsec`字段的值既不是`UTIME_NOW`也不是`UTIME_OMIT`，相应的时间戳设置为相应的`tv_sec`和`tv_nsec`字段。
5. `futimens`需要打开文件更改它的时间
6. `utimensat`可以使用文件名更改时间。
7. `utimes`对路径进行操作。
8. `struct timespec`结构体是：```c
struct {
    time_t tv_sec;
    long tv_nsec;
};
```
9. `struct timeval` 结构体是：```c
struct timeval{
    time_t tv_sec;  //秒
    long tv_usec;   //微妙
};
```

## 读目录
目录可以被任何具有读权限的用户读取，但是为了保护文件系统，只有kernel可以写目录。目录的`w`权限位和`x`权限位表示的是用户是否可以在目录中创建或者删除新文件，并不是说用户可以写目录本身。

给出`opendir`, `fopendir`,
### 函数原型
``` c
#include <dirent.h>

DIR *opendir(const char *name);
DIR *fdopendir(int fd);

struct dirent *readdir(DIR *dirp);
void rewinddir(DIR *dirp);
int closedir(DIR *dirp);
long telldir(DIR *dirp);
void seekdir(DIR *dirp, long loc);
```
### 函数性质
1. `opendir`将文件转换成目录处理函数需要的`DIR`结构，而`fdopendir`将文件描述符转换成目录处理函数需要的`DIR`结构。
2. `telldir`和`seekdir`不是POSIX.1的组成部分，但是它是SUS的XSI扩展。所有UNIX系统都会提供这两个实现。
3. `struct dirent`结构体定义在`<dirent.h>`头文件中，它与实现相关，但是必须包含以下两个成员：``` c
ino_d d_ino;
char d_name[];
```
POSIX.1并没有定义`d_ino`，而POSIX.1的XSI扩展定义了`d_ino`。
4. `DIR`是一个内部结构，这几个函数使用这个内部结构保存当前正在被读的目录的有关信息。`opendir`和`fdopendir`返回`DIR`，而其他函数把`DIR`当做参数。
使用`opendir`打开文件时，`readdir`返回的是目录项中的第一项。
使用`fdopndir`打开文件时，`readdir`返回的第一项取决于传递给`fopendir`的文件描述符的当前文件偏移量。

## 函数`chdir`, `fchdir`和`getcwd`
每一个进程都有一个当前工作目录，**当前工作目录是进程的一个属性，**这个目录是搜索所有相对路径名的起点，不以"/"开始的路径名都是相对路径名。可以使用`chdir`后者`fchdir`改变当前进程的工作目录。它们的原型如下：``` c
#include <unistd.h>

int chdir(const char *path);
int fchdir(int fd);

char *getcwd(char *buf, size_t size);
```
`chdir`的参数是`path`，而`fchidir`的参数是文件描述符。

### `chdir`只会改变当前进程的work dir
**需要注意的一点是，`chdir`和`fchdir`只改变调用这个函数本身的进程，并不影响其他进程。**比如在shell中运行一个程序，在这个程序中更改了进程的当前工作目录，结束这个程序的执行时，shell的当前工作目录并不会改变，因为shell和我们刚才执行的程序属于两个不同的进程。因此，如果要改变shell进程自己的工作目录，应该使用shell直接调用`chdir`函数，所以`cd`命令内建在shell中。
`getcwd`是获得进程当前工作的绝对路径名。内核并不保存目录的完整路径名（linux除外），为了获得进程当前工作的绝对路径名。`getcwd`需要从当前工作目录开始，找到它的上一级目录，读取目录项，找到和工作目录i节点编号相同的目录项，得到对应的文件名。就这样一层一层的向上找，这就找到了绝对路径名。`getcwd`会followsymbolic link，但是不会知道它是由哪里链接到这里的。

### 和进程相关的目录`getcwd`
当一个应用程序需要在经过一些列目录操作之后返回它刚开始的工作目录时。可以先使用`getcwd`获得最开始的工作目录，保存起来，最后再使用`chdir`进行恢复。
`fchdir`可以有更简单的操作，在刚开始时，保存目录的文件描述符。最后使用`fchdir`直接打开这个文件描述符。


## 参考文献
1.《APUE》第三版
2.https://unix.stackexchange.com/questions/21251/execute-vs-read-bit-how-do-directory-permissions-in-linux-work
3.https://superuser.com/questions/168578/why-must-a-folder-be-executable/168583
