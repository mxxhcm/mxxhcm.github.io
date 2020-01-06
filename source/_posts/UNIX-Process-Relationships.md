---
title: Process Relationships
date: 2019-11-28 00:06:11
tags:
 - UNIX
 - 进程
categories: UNIX
---

## 概述
在UNIX Process Control中，介绍了：
1. 每一个进程都有一个父进程，初始的kernel-level的进程通常是它自己。
2. 当子进程终止的时候，父进程可以获得子进程的exit status。
3. 同时在介绍`waitpid`的时候`提到了process groups，并且解释了我们可以等待一个进程组中任意进程的终止。

这一篇文章更详细的介绍了process groups，以及POSIX.1中引入的session的概念。同时还介绍了用于登录的login shell和所有从login shell中启动的进程的关系。

## 终端登录
### BSD登录
系统bootstrap时，内核创建进程号为1的init进程。init进程使系统进行入多用户模式，init读取文件`/etc/tty`，对每一个允许登录的终端设备，init调用一次`fork`，它所生成的子程序`exec getty`程序。
`getty`对终端设备调用`open`函数，以读写方式打开终端。一旦终端被打开，文件描述符0,1,2就被关联到该设备。然后`getty`输出`login`等字样，等待用户输入。当用户输入username之后，`getty`工作就结束了，接下来通过类似于以下的方式调用`login`：```
execle("/bin/login", "login", "-p", username, (char*)0, envp);
```
`login`得到了用户名，接下来调用`getpasswd`提示用户键入密码，然后调用`crypt`将用户键入的口令和shadow中的pwsswd比较，判断密码是否正确。如果密码正确的话，`login`还会进行以下工作：
1. 将当前工作目录改为用户的主目录。
2. 调用`chown`更改终端的控制权，使登录用户成为它的所有者。
3. 对终端设备的权限改成用户读和写。
4. 用`login`得到的所有参数进行初始化
5. `login`进程更改登录用户的`uid`并调用该用户的登录shell。

当然现代的`login`不仅仅进行这些工作，还会根据启动文件更改或者增加用户的环境变量等等。


## 网络登录
### BSD登录
网络登录的话，BSD中有一个inetd进程，等待绝大多数互联想链接。作为系统启动的一部分，`init`调用一个shell，使其执行shell脚本/etc/rc，shell脚本启动一个守护进程inetd。当这个shell脚本终止时，inetd的进程变成init。inetd等待TCP/IP连接，每当有一个连接到达时，就执行一次`fork`，然后使用`exec`执行相应的子程序。
比如一个TELNET服务请求。客服进程打开一个到服务主机的TCP连接，客户机运行TELNET服务进程（用telnetd表示）。它们之间使用TELNET应用协议通过TCP交换数据。客服进程的用户登录到服务进程所在的主机。
然后telnetd进程打开一个伪终端设备，并且使用`fork`将它们分成两个进程。父进程处理通过网络的通信，子进程执行login程序。父进程和子进程之间通过伪终端相连接。在调用`exec`之前，子进程使其文件描述符0,1,2和伪终端相连。登录成功的话，执行和终端操作类似的设置。

当通过终端或者网络登录时，我们得到一个登录shell，它的标准输入，标准输出和标准错误要么连接到一个终端设备，要么连接到一个伪终端设备。

## 进程组
除了pid，每一个进程还属于一个process group。进程组是一个或者一组进程的集合。他们都是同一个job的进程，每一个进程组都有一个唯一的进程组id，和pid类似，可以存放在pid_t中。函数`getpgrp`获得process group的ID，`getpgid`获得指定进程的进程组ID，它们都是SUS定义的。
每个进程组有一个组长进程，组长进程的进程组ID和它的进程ID一样。进程组组长可以创建一个进程组，创建该组中的进程，然后终止。只要在某个进程组中有一个进程存在，那么该进程组就存在，跟其组长是否终止无关。
可以调用`setpgid`创建一个新的进程组后者加入一个现有的进程组。``` c
int setpgid(pid_t pid, pid_t pgid);
```
`setpgid`将pid号为pid的进程的进程组ID设置为pgid。当子进程调用了`exec`之后，父进程就不能修改子进程的进程组ID了。
通常在job control shell中，在fork之后调用此函数，父进程设置子进程的进程组ID，子进程也设置子进程的进程组ID，这两个调用总有一个是重复的，但是可以确保子进程的组ID被正确设置了。

## Session
Session是一个或者多个进程组的集合。比如一个session可以有三个进程组：
进程组1：登录shell，
进程组2：proc1, proc2
进程组3：proc3, proc4, proc5
等等。通常一个进程组的进程是由一个shell pipeline生成的。比如上面的进程组可能是通过以下shell命令实现的：
proc1 | proc2 &
proc3 | proc4 | proc5

可以调用`setsid`创建一个新的session：``` c
#include <unistd.h>

pid_t setsid(void);
```
这个函数具有以下性质：
如果调用这个函数的进程不是一个进程组的组长，就创建一个新的session：
1. 该进程变成新的session的session leader，这个session leader是创建该session的进程。注意SUS只说明了session leader，而没有像pid和process gid之类的session id。也就是说session leader是有唯一PID的单个进程，可以将session leader的ID当做session ID。**注意什么是session leader，它是一个进程，而session ID是session leader的PID，或者也把session ID较为session leader的process group ID**。
2. 调用进程是新进程组和新session中的唯一一个进程。
3. 新的session没有controlling terminal。


## 控制终端
session和process group的一些其他属性：
- 一个session通常会有一个controlling terminal，通常是终端设备或者伪终端设备。
- 建立和控制终端连接的session leader被称为controlling process（控制进程）。
- 一个session中的几个process group可以被分为一个foreground process group（前台进程组）和多个background process group（后台进程组）。
- 如果一个session有一个controlling terminal，那么它有一个前台进程组，其它进程组为后台进程组。
- 无论何时键入终端的中断键，ctrl+C，都会将中断信号发送至前台进程组的所有进程。
- 无论何时键入终端的退出键，ctrl+\，都会将退出信号发送到前台进程组的所有进程。
- 如果终端接口检测到网络已经断开，将挂断信号发送到session leader。

登录shell属于后台进程组，它是session leader，也就是controlling process。登录时，会自动建立controlling terminal。有时候不管标准输入，标准输出是否重定向，程序都要和控制终端交互，可以open文件/dev/tty。在内核中，/dev/tty是controlling terminal的同义词，如果没有controlling terminal，对于这个设备的open失败。

## `tcgetpgrp`, `tcsetpgrp`和`tcgetsid`
### 函数原型
``` c
#include <unistd.h>

pid_t tcgetpgrp(int fd);
int tcsetpgrp(int fd, pid_t pgrp);

#include <termios.h>

pid_t tcgetsid(int fd);
```
### 属性
1. `tcgetpgrp`返回前台进程组ID，它与在fd上打开的终端相关联。
2. 如果进程有一个controlling terminal，这个进程可以调用`tcsetpgrp`将前台进程组ID设置为pgrpid，fd必须引用该session的controlling terminal。
3. 可以通过`tcgetsid`函数获得session leader的进程组ID。

## job control
可以在一个终端上启动多个jobs（groups of process），它控制哪个job可以访问终端，哪个job应该在后台运行，job control需要满足以下三个条件：
1. 支持job control的shell
2. 内核的终端驱动程序必须支持job contrl
3. 内核必须支持某些特定的job-control signals。

在shell中使用job contrl，我们可以创建前台的job，也可以创建后台的job，一个job是进程的集合，通常是进程的pipeline，可以在后台运行多个job。
可以通过键入几个特殊字符和终端驱动程序进行交互作用，控制前台进程组的所有进程：
- 中断，ctrl+C，产生SIGINT
- 退出，ctrl+\，产生SIGQUIT
- 挂起，ctrl+Z，产生SIGSTRP

终端驱动程序还需要处理一些情况：
1. 当后台job试图读取终端时
如果有一个前台job和多个后台jobs。一般情况下，只有前台job接收终端输入，当后台job试图从终端读取，并不会报错，终端驱动程序会检测这种情况，并且向后台job发出一个SIGTTIN signal。这个signal会停止后台job，shell向有关用户发出通知说你的后台job停止啦！然后用户可以用shell命令将它转换后前台job，从终端读取。
如果
2. 当后台job试图写终端时
当用户禁止后台job向controlling terminal写后，当后台job试图写向标准输出，终端驱动程序识别出这个写操作来自于后台job，向该job发出SIGTTOU signal，阻塞相应的job。当用户使用fg将后台job转换为前台job时，job继续执行。


## shell执行过程
有些shell支持job control，比如bash，有些不支持，比如Bourne shell。

执行以下命令：``` shell
ps -o pid,ppid,pgid,sid,comm | cat      // 后台job
ps -o pid,ppid,pgid,sid,comm | cat &    // 前台job
```

### 不支持job control的shell
在不支持job control的shell中，管道的最后一个进程是shell的子进程，而执行管道中其他命令的进程是该最后进程的子进程。当最后一个进程终止时，shell得到通知。
所有的job的process group id和shell的都一样。
![shell_no_job_control](shell_no_job_control.png)

### 支持job control的shell
而在支持job control的shell中
每一个job都有一个自己的process group id，和shell的不一样。
shell是两个job的父进程。


## 孤儿进程组
当一个进程的父进程退出之后，而子进程还没有结束，这个进程就成了孤儿进程。进程组也可以是孤儿进程组。
什么是孤儿进程组：
进程组中每个成员的父进程要么是它组内的一个成员，要么不是这个进程组所在session的成员。

## FreeBSD实现
每个session都会有一个seesion结构，它包含：
- `s_count`
- `s_leader`
- `s_ttyvp`
- `s_ttyp`
- `s_sid`，这一部分不是SUS的组成，只有FreeBSD有。

每个终端或者伪终端会在内核中分配一个tty结构，它包含：
- `t_session`
- `t_pgrp`
- `t_termios`
- `t_winsize`

每个进程组都包含一个pgrp结构，它包含：
- `pg_id`
- `pg_session`
- `pg_memebers`

每个进程都有一个`proc`结构，它包含：
- `p_pid`
- `p_ptr`
- `p_grp`
- `p_pglist`

进程通过v_node结构体访问/dev/tty。

它们之间的关系如下图所示：
![session_and_process_group]()

## 参考文献
1.《APUE》第三版
