---
title: UNIX Process Control
date: 2019-11-25 10:31:02
tags:
 - UNIX
 - 进程
categories: UNIX
---

## 概述
这一节主要介绍UNIX系统的进程控制，包括进程创建，进程执行和进程控制。以及进程属性的各种的ID-real UID, real GID和effective UID, effective GID和save UID， set UID和set GID，以及它们如何收到进程控制原语的影响。


## 进程标识(pid)
每一个进程都有一个非负整数表示它的唯一进程ID。进程ID标识符总是唯一的，但是可以复用。
系统中有一些专用进程，具体细节跟实现有关。
获得当前进程的各项id：
``` c
// real uid
uid_t ruid = getuid()
// effective uid
uid_t euid = geteuid()

// real gid
uid_t rgid = getgid()
// effective gid
uid_t egid = getegid()

// pid
uid_t pid = getpid()
// parent pid
uid_t ppid = getppid()
```

## `fork`
`fork`创建一个子进程。函数原型：

### `fork`原型
```c
#include <unistd.h>

pid_t fork(void);
```

### `fork`性质
1. **进程ID**。`fork`调用一次，返回两次，分别是0和子进程ID，用以区别父进程和子进程。对于父进程，返回子进程ID，对于子进程，返回0。因为父进程可能有多个子进程，并且没有提供获得一个进程所有子进程ID的函数，而fork只有一个父进程，可以通过`getppid`获得它的父进程的ID。所以这样子进行区分。
2. 子进程和父进程分别继续执行调用`fork`之后的指令。子进程是父进程的副本。子进程获得父本的数据段，堆和栈的完全副本。这是子进程的副本，和父进程不一样，它们并不共享数据的内存空间，但是它们共享text segment。
3. 现代的操作系统实现，使用写时复制代替了父进程数据段，堆和栈的完全副本。这些区域是由父进程和子进程共享的，但是它们的访问权限是只读。如果父进程或者子进程想要对这些区域进行修改的话，内核会为修改区域的那块内存制作一个副本，用于进程修改。
4. 父进程和子进程因为不共享数据，堆和栈，每个进程都有自己的变量，不会相互影响。
5. **执行顺序**。`fork`后父进程和子进程的执行顺序是不确定的，这跟内核的调度算法有关。如果要求父进程和子进程之间进行同步，需要它们之间进行某种形式的进程通信。
6. **文件共享**。对于父进程打开的文件，`fork`相当于将父进程的文件描述符都复制到了子进程中，相当于对父进程的每一个文件描述符，都调用了`dup`函数。父进程和子进程每个相同的打开文件描述符共享同一个文件表项。一般来说，在`fork`之后处理文件描述符有以下两种情况：
    - 父进程等待子进程完成。父进程不需要对它的文件描述符做任何处理。
    - 父进程和子进程分别执行不同的程序段。父进程和子进程各自关闭它们不需要的文件描述符。
7. **`fork`后子进程继承的信息**。
    - real UID, real GID, effective UID, effective GID 
    - set UID和 set GID
    - 附属组ID
    - 进程组ID
    - Session ID
    - 控制终端
    - cwd
    - root dir
    - umask
    - signal mask
    - 文件描述符标志
    - 环境
    - 共享的内存段
    - 内存映像
    - Resource limits
8. **父进程和子进程的区别。**
    - `fork`的返回值不同
    - pid不同
    - 它们有不同的ppid
    - 子进程的很多时间设置为0
    - 父进程设置的文件锁子进程不继承。
    - 子进程的未处理闹钟被清除
    - 子进程的未处理信号被设置为空集。
9. **`fork`的两种用法**。
    - 父进程和子进程分别执行不同的代码。比如网络服务中，父进程负责等待客户端请求，子进程负责处理父进程接收到的请求。
    - 一个进程要执行不同的程序。对shell比较常见，通常执行完`fork`返回后立即调用`exec`。

## `vfork`
创建一个子进程，并且阻塞父进程，函数原型如下：

### `vfork`原型
``` c
#include <sys/types.h>
#include <unistd.h>

pid_t vfork(void);
```

### `vfork`属性
1. `vfork`和`fork`都创建一个新进程，但是`vfork`并不会将父进程的地址空间完全复制到子进程中。因为子进程会立即调用`exec`或者`exit`，就不会引用该地址空间。但是如果在调用`exec`或者`exit`之前，它会在父进程的空间中运行，这种做法会提高效率。但是如果子进程修改除了`vfork`的返回值，或者在没有调用`exit`或者`exec`之前调用其他函数，这种行为是未定义的。
2. `vfork`保证子进程先运行，在子进程没有调用`exec`或者`exit`时，内核会使父进程休眠，在子进程调用`exec`或者`exit`之后父进程才会恢复运行。如果子进程需要父进程进一步操作的时候，就会产生死锁。

## `exit`函数
总共有八种方式可以让进程终止，包括五种正常和三种异常，前五种是正常终止，后五种是异常终止：
1. 从`main`返回，相当于调用`exit`。
2. 调用`exit`，ISO C定义的，它的操作包括调用各个exit handler，处理所有标准I/O流。
3. 调用`_exit`或者`_Exit`，ISO C定义了`_Exit`，而POSIX.1说明了`_exit`。它的目的是提供一种无需运行exit handler或者信号处理程序而终止的方法。是否对标准I/O流进行flush，取决于实现。在UNIX中，`_Exit`和`_exit`是同义的，并不flush I/O流。
4. 最后一个线程从其启动例程返回
5. 最后一个线程调用`pthread_exit`
6. 调用`abort`
7. 接到一个`signal`
8. 最后一个线程对取消请求做出响应

不管进程以哪种方式终止，最后都会执行内核中的同一段代码，这段代码为相关进程关闭所有打开的文件描述符，释放它使用的内存。
为了让终止进程能够通知父进程它是如何终止的。对于3个终止函数，将它的`exit status`作为参数传递给函数。在异常终止的情况下，内核产生一个指示其异常终止原因的terminaiton status（终止状态）。在任意终止情况下，这个终止进程的父进程都能用`wait`或者`waitpid`函数获得它的终止状态。
**如果父进程在子进程之前终止**，那么对于父进程终止的所有进程，它们的父进程都变成`init`进程，终止状态返回到`init`进程。具体是怎么操作：对于一个即将终止的进程，内核检查所有活动进程，判断其中是否有待终止进程的子进程，如果有的话，将这些进程的父进程的ID改为`init`进程的ID 1。
**如果子进程在父进程之前终止**，那么父进程是无法获取它的终止状态的。内核为每一个终止进程保留了一部分信息，当终止进程的父进程调用`wait`或者`waitpid`时，可以获取这些信息，这些信息包含终止进程PID，进程的终止状态，进程占用的CPU时间总量。内核可以释放这些进程的内存，关闭打开的文件。如果一个进程终止了，但是它的父进程没有等待它，它被称为一个zombie（僵尸）进程。如果一个长期运行的进程，`fork`了很多子进程，除非父进程等到取得子进程的终止状态，要不它们就会变成僵尸进程。**当父进程结束时，僵尸进程就会结束？？？**
`init`的子进程，不会变成僵尸进程，因为`init`进程被编写成无论何时只要有一个子进程终止，`init`就会调用一个`wait`函数获得其终止状态。

## `wait`和`waitpid`
当一个进程终止的话（无论正常还是异常），内核就会向它的父进程发送`SIGCHLD`信号。而子进程终止是个异步事件，可以在父进程运行的任何时候发生。对于这种信号，父进程可以忽略它，或者调用一个信号处理函数。
调用了`wait`或者`waitpid`的进程，可能会处于以下几种状态之一：
1. 所有子进程都还在运行，则阻塞。
2. 一个子进程已经终止，正在等待父进程获取其终止状态，那么取得该子进程的终止状态立即返回。
3. 如果它没有任何子进程，立即出错返回。

如果进程由于接收到`SIGCHLD`而调用`wait`，我们期望`wait`会立即返回。如果在随机的时间点调用`wait`，进程可能会阻塞。

### `wait`和`waitpid`原型
``` c
#include <sys/wait.h>

pid_t wait(int *wstatus);
pid_t waitpid(pid_t pid, int *wstatus, int options);
```

### `wait`和`waitpid`性质
1. 在一个子进程终止前，`wait`使其调用者阻塞，直到任意一个子进程终止。`wait`返回终止子进程的进程ID。如果`wait`要等待一个特定的进程，将返回的pid和要等待的pid相比，如果不相等，将这个pid和termination status保存起来，再次调用`wait`，直到等到目标pid。下一次想要等待一个特定进程的时候，现场看已经终止的进程列表中是否已有它 ，没有的haunted继续调用`wait`。
2. `waitpid`可以指定等待的进程的pid。当`pid`为-1时，`waitpid`和`wait`一样。当`pid`大于或者小于0时，等待相应的pid（绝对值）。当pid等于0时，等待gid等于调用进程组id的任意一个子进程。
3. 而`waitpid`可以设置使得调用不阻塞。
4. `waitpid`通过`WUNTRACED`和`WCONTINUED`选择支持job control。
5. 对于`wait`，只有当调用进程没有子进程时，才出错。对于`waitpid`，指定的进程或者进程组不存在，或者参数pid不是调用进程的子进程时，都会出错。
6. `wstatus`是一个整形指针。如果它不为空指针，终止进程的终止状态就存放在它所指的单元内。
7. `wstatus`指向的整形变量的意义是由实现定义的，其中的某一些位表示exit status，即正常退出。另外一些位表示signal number，表示不正常退出，一位表示是否产生core file，等等。POSIX.1指定了termination status可以用`<sys/wait.h>`中定义的宏查看。四个互斥宏可以用来取得进程终止的原因：
    - `WIFEXITED(status)`，如果status是一个正常终止子进程返回的，为true。执行`WEXITSTATUS(statue)`获取子进程传递给`exit`或者`_exit`的参数的低八位。
    - `WIFSIGNALED(status)`，如果status是一个异常终止子进程返回的，为true。执行`WTERMSIG(status)`获取使得子进程终止的signal。
    - `WIFSTOPPED(status)`，如果status是一个当前暂停的子进程返回的，为true。执行`WSTOPSIG(status)`获取使得子进程暂停的signal。
    - `WIFCONTINUED(status)`，。
8. `fork`两次可以让原始进程不用自己调用`wait`，也可以避免产生僵尸进程。

## `waitid`
### `waitid`原型
``` c
#include <sys/wait.h>

int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
```

### `waitid`性质
1. `waitid`是SUS指定的，不是ISO C的部分。
2. `waitid`和`waitpid`类似，但是它使用两个单独的参数表示要等到的子进程所属的类型，用`idtype`表明`id`的类型，用`id`表示pid或者进程组id。`idtype`的取值如下：
    - `P_PID`，等待特定进程，id指定要等待的子进程的pid
    - `P_PGID`，等待特定进程组中的任一子进程，id是包含要等待子进程的组ID
    - `P_ALL`，等待任意子进程，忽略id。
3. `options`参数是以下标志的按位或运算。
    - `WCONTINUED`
    - `WEXITED`
    - `WNOHANG`
    - `WNOWAIT`
    - `WSTOPPED`
4. `siginfo_t`结构体包含了子进程状态改变有关signal的详细信息。

## `wait3`和`wait4`
大多数UNIX系统都支持`wait3`和`wait4`，它们是从BSD延续下来的。它们的功能比POSIX.1函数`wait`, `waitpid`和`waitid`要多一个。可以通过附加参数允许内核返回终止进程以及其所有子进程使用的资源概况。包含用户CPU时间总量，系统CPU时间总量，缺页次数，接收到的signal次数。
它们的原型如下：

### `wait3`和`wait4`原型
``` c
#include <sys/types.h>
#include <sys/time.h>
#include <sys/resource.h>
#include <sys/wait.h>

pid_t wait3(int *wstatus, int options, struct rusage *rusage);
pid_t wait4(pid_t pid, int *wstatus, int options, struct rusage *rusage);
```

## race condition
如果多个进程都企图对共享数据进行某种处理，而且最后的结果取决于进程运行的顺序时，我们认为发生了race condition。
如果一个进程希望等待子进程终止，可以调用`wait`函数中的一个。如果一个子进程想要等待父进程终止，可以使用下列形式的循环，称为轮询（polling)```c
while(getppid() != 1)
{
    sleep();
}
```
这种方式浪费了很多CPU时间。为了避免这些问题，可以使用signal或者进程间通信解决这些问题。

## `exec`
通常使用`fork`创建新的子进程之后，子进程往往会调用一种`exec`函数执行另一个程序。当进程调用`exec`函数时，该进程执行的程序完全替换为新程序，而新程序从其`main`函数开始执行。调用`exec`并不会创建新进程，所以前后的进程ID不变。`exec`使用磁盘上的一个新程序替换了当前进程的text segment, data segment, heap和stack。`exec`函数只有在出错的时候才返回-1，并且设置`errno`。
总共有七种不同的`exec`函数，它们被统称为`exec`函数。它们的原型如下：

### `exec`函数原型
``` c
#include <unistd.h>

extern char **environ;

int execl(const char *path, const char *arg, ... /* (char  *) NULL */);
int execv(const char *path, char *const argv[]);
int execle(const char *path, const char *arg, ... /*, (char *) NULL, char * const envp[] */);
int execve(const char *filename, char *const argv[], char *const envp[]);

int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[],
int fexecve(int fd, char *const argv[], char *const envp[]);
```

### `exec`函数属性
1. **传入参数的区别。** `path`是路径名作为参数，`file`是文件名作为参数。如果`path`中包含`'/'`，将它看成路径名。否则按照PATH环境变量，在它指定的目录中搜寻可执行文件。如果函数`execlp`和`execvp`在PATH指定的目录中找到的文件不是link editor产生的可执行文件，就会把它当做一个shell脚本，调用`/bin/sh`，把这个文件当做shell的输入。
2. **argmuent list的区别。** l表示的是列表，v表示的是向量。`execl`, `execlp`和`execle`要求将新程序的每个命令行参数都说明为一个单独的参数，这种参数表以空指针结尾。而`execv`,`execvp`, `execve`和`fexecve`，需要先构造一个指向各个参数的指针数组，然后将该数组的地址作为这四个函数的参数。
3. **environment list的区别。** 以e结尾的函数，`execve`, `execvpe`,`execle`, `fexecve`等可以传递一个指向environment字符串指针数组的指针，这个是自己指定的环境。其他几个不带e的函数使用进程中的environ变量为新程序复制现有的环境。
4. 调用`exec`后，进程ID没有改变，但是新程序从调用进程继承了以下属性：
    - pid和ppid
    - real UID, real GID, 
    - 附属组ID
    - 进程组ID
    - Session ID
    - 控制终端
    - 闹钟余留时间
    - cwd
    - root dir
    - umask
    - 文件锁
    - 进程信号屏蔽
    - 未处理信号
    - 资源限制
    - nice值
    - 时间
5. 在`exec`前后，real UID和real GID不变，effective ID取决于是否设置set UID和set GID。如果新程序的set UID已经设置，则effective ID变成程序文件所有者的ID，否则不变。
6. 这几个函数中，只有`execve`是系统调用，其他几个都只是库函数，最终都要调用`execve`。

## 更改UID和GID
进程的real UID, real GID以及effective UID，effective GID都是可以改变的。可以使用`setuid`, `seteuid`和`setgid`, `setegid`更改real UID， effective UID，和real GID以及effective GID。它们的原型如下：

### `setuid`, `seteuid`和`setgid`, `setegid`原型
``` c
#include <unistd.h>

int setuid(uid_t uid);
int setgid(gid_t gid);
int seteuid(uid_t euid);
int setegid(gid_t egid);
```

### `setuid`, `seteuid`和`setgid`, `setegid`属性
更改进程的UID需要遵守以下规则：
1. 如果进程具有超级用户权限。`setuid`函数将real UID, effective UID和saved set UID都保存为uid。
2. 如果进程没有超级用户权限。但是uid等于real UID或者saved set UID，则`setuid`只将real UID改成uid。其他UID不变。
3. 如果两个条件都不满足，设置`errno`为`EPERM`，返回-1。
4. 只有superuser进程可以改变real UID。一般来说，real UID是用户登录时，login程序设置的，而且不会改变它。login是一个超级用户进程，当它调用`setuid`时，设置所有三个uid。
5. 只有set-user-ID位被设置时，`exec`才设置effective UID。如果set-user-ID位没有被设置，`exec`不修改effective UID。任何时候，都可以调用`setuid`将effective UID设置为real UID和saved set UID修改为。
6. saved set-user-ID是`exec`从effective UID复制而来的。如果设置了set-user-ID位，在`exec`根据文件的UID设置了进程的effective UID之后，这个副本就被保存起来了。

总结得到以下的表格：
|ID| |`exec`| |`setuid`|
|:--:|:--:|:--:|:--:|:--:|
||set-user-ID位关闭|set-user-ID位开启| 超级用户 |  非超级用户|
|real UID|不变|不变|设置为uid|不变|
|effective UID|不变|程序文件的UID|设置为uid|设置为uid|
|saved set-UID|从effective ID复制|从effective ID复制|设置为uid|不变|

### `setreuid`和`setregid`
``` c
#include <unistd.h>

int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);
```
1. 交换real UID和effective UID。一个非root用户总能交换real UID和effective UID。允许一个set-user-ID程序交换成用户的普通权限，然后再次交换回去。
2. 某个参数设置为1，表示相应的ID不变

### `seteuid`和`setegid`
``` c
#include <sys/types.h>
#include <unistd.h>

int seteuid(uid_t euid);
int setegid(gid_t egid);
```
1. 一个非root用户可以将effective UID设置为real UID或者saved set-UID。
对于一个特权用户，可以将effective UID设置为uid。和`setuid`的区别在于，它修改三个UID。

### 组ID
修改进程的GID和修改进程的UID类似。附属组ID不受`setgid`，`setegid`和`setrugid`的影响。

## 解释器文件
所有的UNIX系统都支持解释器文件。这种文件是文本文件，起始形式是：``` txt
#! pathname [optional-argument]
```
在感叹号和pathname之间的空格是可以选的。常见的解释器文件以下列行开始：```shell
#! /bin/bash
```
pathname通常是绝对路径名。对这种文件的识别是由内核作为`exec`系统调用处理的一部分完成的。内核是调用`exec`函数的进程实际执行的并不是该解释器文件，而是在该解释器文件第一行中pathname指定的文件。解释器文件是文本文件，它以!#开头，而解释器是二进制文件，由解释器中的文件第一行的pathname指定。

是否一定需要解释器文件呢？不完全如此，但是它们确实使得用户得到效率方面的好处，代价是内核的额外开销，因为识别解释器文件的是内核。由于下列原因，解释器文件是有用的：
1. 有些程序是用某种脚本语言写的脚本，解释器文件可以将这一事实隐藏起来。
2. 解释器脚本在效率方面提供了好处。
3. 解释器脚本使我们可以使用除了/bin/sh以外的其他shell编写shell脚本。

## `system`
ISO C定义了`system`函数，它用来执行一个shell命令，对系统的依赖性很强。`system`库函数调用`fork`创建一个子进程使用`execl`执行参数`system`指定的shell命令。相当于:```c
execl("/bin/sh", "sh", "-c", command, (char *) 0);
```
POSIX.1包括了`system`接口，扩展了ISO C定义，描述了`system`在POSIX.1环境中的运行行为。
```c
#include <stdlib.h>

int system(const char *command);
```
`system`在其实现中调用了`fork`, `exec`和`waitpid`，可能有以下的返回值：
1. 如果`command`是一个空指针，当前系统是有可用的shell时，`system`返回非0值，否则返回0。在UNIX的各个实现中，一定提供了shell，当`command`是空指针时，总是返回非零值。
2. `fork`失败或者`waitpid`返回处`EINTR`之外的出错，返回-1，设置errno。
3. 如果`exec`失败，表示不能执行shell，返回值如同shell执行了exit(127)一样
4. 所有三个函数都成功，`system`的返回值和在shell中执行相应命令的的termination status一样。

其中一种`system`的可能实现如下所示：```c
int system(const char *cmdstring)
{
    pid_t pid;
    int status;

    if(cmdstring == NULL)
        return 1;
    
    if ((pid = fork()) < 0)
    {
        status = -1;
    }
    else if(pid == 0)
    {
        execl("/bin/sh", "sh",  "-c", cmdstring, (char*)0);
        _exit(127);
    }
    else
    {
        while(waitpid(pid, &status, 0) < 0)
        {

            if(errno != EINTR)
            {
                status = -1;
                break;
            }
        }
    }

    return status;
}
```
使用`system`而不是直接使用`fork`和`exec`的好处是：`system`进行了各种所需要的各种出错处理和信号处理。但是早期的系统中，没有`waitpid`函数，于是父进程使用下列语句等待子进程结束：``` c
while((lastpid = wait(&status) != pid && lastpid !=-1)
    ;
```
如果在调用`system`之前产生了其它子进程，如果这些子进程在`system`产生的子进程之前结束，那么上面的代码会将这些提前结束的子进程的进程ID和termination都给丢弃。
**system函数还有可能会出现漏洞。如果设置了set UID或者set GID位的程序执行`system`，那么这个进程的高级别权限可能会保持下来（现代的系统都解决了这个问题）。如果一个进程正在以特殊的权限(set UID和set GID)运行，它又想生成另一个进程执行另一个程序，它应该直接使用`fork`和`exec`，而且在`fork`之后，`exec`之前要改回普通权限，set UID和set GID程序绝不应该调用`system`函数。**

## 进程会计
大多数UNIX系统都提供了一个选项进行进程会计处理。启动该选项之后，每当进程结束时内核就会写一个会记记录。典型的会计记录是一个二进制数据，一般包括命令名，所有的CPU时间总量，UID和GID，启动时间等。所有的标准都没有定义进程会记，所以实现上就千差万别。
`acct`函数启用和关闭进程会计。
会记记录结构定义在头文件`<sys/acct.h>`的`struct acct`中，其中`ac_flag`标志记录了进程执行期间的某些事件：
- `AFORK`，进程是`fork`产生的，但是未调用`exec`
- `ASU`，进程使用superuser权限
- `ACORE`，进程转储到core
- `AXSIG`，进程由一个signal杀死
- `AEXPND`，扩展的会计条目
- `ANVER`，新纪录格式

在LINUX上，`ac_flag`是枚举类型，所以不能使用`#ifdef`判断是否支持`ACCORE`等flag，可以使用`if !defied HAS_ACCORE`进行判断。

会计记录所需的各个数据（各CPU时间，传输的字符数等）都由内核保存在process table中，并在一个新进程被创建时初始化，进程终止时写一个会计记录。这产生了两个后果：
1. 对于那些不会终止的进程，比如`init`进程，我们无法获得它的会计记录。内核守护进程也不会终止，所以也不会产生会计记录。
2. 在会计文件中记录的顺序对应于进程终止的顺序，而不是它们启动的顺序。

会记记录对应于进程而不是程序。在`fork`之后，内核为子进程初始化一个记录，而不是在一个新程序被执行初始化时。`exec`并不会创建一个新的记录，但是相应记录中的名字会改变，`AFORK`标志没了。

## 获得当前登录用户名
可以使用`getlogin`获得当前登录用户的用户名。
``` c
#include <unistd.h>

char *getlogin(void);
```

## 进程调度
调度策略和调度优先级是由内核确定的。

### `nice`, `getpriority`, `setpriority`原型
``` c
#include <unistd.h>

int nice(int inc);

#include <sys/resource.h>

int getpriority(int which, id_t who);
int setpriority(int which, id_t who, int prio);
```

### `nice`, `getpriority`, `setpriority`属性
1. `nice`函数将输入的参数加到当前的`nice`值上。`nice`值越大，优先级越低，否则越高。
2. 在单核的机器中，同时运行一个父进程和一个子进程，它们的`nice`值不同的话，CPU占用比也可能会不同，这取决于进程调度程序如何使用`nice`值。在多核的机器上可能看不到这样的结果。

## 进程时间
可以使用`times`获得某进程和它的子进程的CPU时间以及墙上时钟时间，`times`通过`struct tms`传递信息。它的内容如下：``` c
struct tms {
    clock_t tms_utime;
    clock_t tms_stime;
    clock_t tms_cutime;
    clock_t tms_cstime;
};
```
它包含进程的用户CPU时间，系统CPU时间和子进程的用户CPU时间和系统CPU时间。但是不包含墙上时钟时间，墙上时钟时间是通过函数的返回值得到的，而且得到的时间是相对于过去某个时间点得到的，所以不能使用它的绝对值，要使用相对值。比如第一次调用`times`，记录返回值，等到下一次调用`times`时，用新的值减去刚才保存的值，得到墙上时间时间。
**所有时间（结构体和返回值）的单位都是滴答数。**
函数原型如下：``` c
#include <sys/times.h>

clock_t times(struct tms *buf);
```
shell的`time(1)`可以使用`times(2)`实现，看程序。


## 参考文献
1.《APUE》第三版

