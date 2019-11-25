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
1. `fork`调用一次，返回两次，分别是0和子进程ID，用以区别父进程和子进程。对于父进程，返回子进程ID，对于子进程，返回0。因为父进程可能有多个子进程，并且没有提供获得一个进程所有子进程ID的函数，而fork只有一个父进程，可以通过`getppid`获得它的父进程的ID。所以这样子进行区分。
2. 子进程和父进程分别继续执行调用`fork`之后的指令。子进程是父进程的一副本，，


## `vfork`

## `exit`

## `wait`和`waitpid`

## `waitid

## `wait3`和`wait4`

## race condition

## `exec`

## 更改UID和GID

## 解释器文件

## `system`

## 进程会计

## 用户标识

## 进程调度

## 进程时间

## 参考文献
1.《APUE》第三版

