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
文件类型信息包含在`struct stat`的`st_mode`字段中。UNIX系统的文件类型有以下几种：
1. 普通文件，可以是二进制文件，也可以是文本文件。除了二进制可执行文件必须遵循标准化格式外，其他的文件对于UNIX内核来说基本上没有区别。使用`S_ISREG`宏进行判断。
2. 目录文件，包含了其他文件的名字，以及指向这些文件有关信息的指针。对一个目录具有读权限的任意进程都可以读目录的内容，但是只有内核才可以直接写目录文件。使用`S_ISDIR`宏进行判断。
3. block special file，提供对设备带缓冲的访问，每次访问以固定的长度进行。使用`S_ISBLK`宏进行判断。
4. character special file，提供对设备不带缓冲的访问，每次访问长度可变。系统中的设备要么是block special file要是character special file。使用`S_ISCHR`宏进行判断。
5. FIFO，用于进程间通信。使用`S_ISFIFO`宏进行判断。
6. socket，用于进程间的网络通信。使用`S_ISSOCK`宏进行判断。
7. sysbolic link，指向另一个文件。使用`S_ISLNK`宏进行判断。

## 和进程相关的UID和GID
每一个进程有6个或者更多和它相关的ID：
### 实际UID和GID
real user ID和real group ID，用来表示当前用户。

### 有效UID和GID
effective user ID和effective group ID，决定我们的文件访问权限。通常情况下，effective user ID以及effective group ID和real user ID以及real group ID一样。

### Set-User-ID和Set-Group-ID
saved set-user-ID和saved set-group-ID，在执行一个程序时，包含了有效user ID和有效group ID的副本。
每一个文件都有一个所有者和组所有者，它们的值在`st_uid`和`st_gid`中。
当执行一个程序文件时，进程的effective user ID通常就是real user ID，而effective group ID通常就是real group ID。但是可以在`st_mode`中设置一个特殊的flag，意思是当执行此文件时，将执行此文件的进程的effective user ID设置为文件所有者的user ID。同样，还有另一个特殊的flag，它将执行此文件的进程的effective group ID设置为文件组所有者的user ID。这两个标志位被记为set-user-ID bit和set-group-ID bit，它们都存放在`st_mode`中，可以使用`S_ISUID`和`S_ISGID`测试。
**运行set UID程序的进程通常会获得额外的权限！！！所以要格外注意。**

## 文件和目录的访问权限
`st_mode`中还包含了文件的访问权限。对于所有文件类型（不单单是文件和目录），都有三种访问权限：

### r-读权限
读权限查询文件名数据

### w-写权限
- 新建文件与目录 
- 删除文件或者目录 
- 重命名以及转移文件或者目录
	
### x-可执行权限
- 进入某目录 
- 切换到该目录（cd命令）

!!!能不能进入某一目录只与该目录的x权限有关，如果不拥有某目录的x权限，即使拥有r权限，那么也无法执行该目录下的任何命令
但是即使拥有了x权限，但是没有r权限，能进入该目录但是不能打开该目录，因为没有读取的权限。

### 九个访问权限位
将`rwx`和user, group以及other进行组合，总共有九个访问权限位：
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
进程每次打开，创建或者删除一个文件时，内核就会进行文件访问权限测试，这种测试可能涉及到文件的所有者ID，文件的组所有者ID，进程的effective user ID和effective group ID。文件的所有者ID和文件的组所有者ID都是文件的属性，而effective user ID和effective group ID是进程的属性。
内核进行访问权限测试的步骤如下：
1. 如果进程的effective user ID是0，结束权限判断，允许各项访问。否则跳转第2步进行判断。
2. 如果进程的effectiev user ID等于文件所有者ID，也就是`st_uid`，结束权限判断，根据访问权限位允许相应操作。否则跳转第3步。
3. 如果进程的effective group ID等于文件的group ID，结束权限判断，根据访问权限的设置允许相应的操作，否则跳转第4步。
4. 如果不满足前三条，就按照若其他用户的访问权限位判定操作是否合法。

总结一下，就是依次判断effective user ID是不是等于root，effective user ID是不是等于`st_uid`，或者effective group ID是不是等于`st_gid`，如果都不满足，就按照其它权限判定当前进程对文件的操作是否被允许。按照顺序来判断，满足一个就不用判断后面的了。

## 新文件和新目录的所有权
新文件的user ID设置为进程的effective user ID。关于新文件的group ID，可以选择以下两种方式中的一个进行设置：
1. 新文件的group ID可以是进程的effective group ID
2. 新文件的group ID可以是它所在目录的group ID。

不同的UNIX实现有不同的设置，这里拿linux来说，Linux 3.2.0以后，新文件的group ID取决于它所在目录的set-group ID bit是否被设置，如果被设置了，新文件的ID就是它所在目录的GID，否则就是进程的effective GID。

## `access`和`faccessat`
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

## 函数`umask`
前面介绍了和文件相关的9个访问权限位，在此基础上可以使用和每个进程相关的file mode创建mask。`umask`函数为进程设置mask，原型如下：

### 函数原型
``` c
#include <sys/stat.h>

mode_t umask(mode_t cmask);
```

### 特点
1. 这个函数的作用是去掉`cmask`中指定的权限，返回之前的mode。
2. **在程序中创建新文件时，如果想要确保指定的访问限权激活，必须在进程运行时修改`umask`的值。否则，`umask`可能会覆盖掉我们创建文件时指定的权限位。**
3. shell中有内置的`umask`命令，SUS要求shell的`umask`除了支持八进制的拒绝权限外，还要支持符号格式的指定许可的权限。使用``` shell
umask -S
```
查看。

## 函数`chmod`, `fchmod`和`fchmodat`
文件的访问权限可以使用`chmod`, `fchmod`和`fchmodat`进行修改，它们的原型如下：
#### 原型
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
4. 当flags设置了`AT_SYMLINK_NOFLOLLW`时，不会follow符号链接。
5. 在以下两种情况下，`chmod`函数自动清除两个权限位：
- 新创建文件的GID可能不是调用进程的effective GID。新文件的GID可能是父目录的GID。如果新文件的GID不等于进程的effective GID，而且进程没有root权限，set-group-id位会被自动关闭。
- stick bit的设置

## sticky bit

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
