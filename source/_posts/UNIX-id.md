---
title: UNIX id
date: 2020-02-20 15:03:12
tags:
 - UNIX
categories: UNIX
---

## 和进程相关的ID
1. 每个进程都有一个进程ID(pid)，还有一个父进程ID(ppid)。
除此之外，每个进程还有以下ID:
2. 实际用户ID，实际组ID。实际用户ID和实际组ID就是当前用户的UID和GID。
3. 有效用户ID，有效组ID，附属组ID。通常情况下，实际用户ID和有效用户ID是一样的，实际组ID和有效组ID也是一样的。
4. 保存的设置用户ID，保存的设置组ID，它们包含了执行一个程序时有效U(G)ID的副本。可以在st_mode中设置set-user-ID和set-group-ID位（这个和保存的设置U(G)ID不是一个东西），表示，当执此文件时，将进程的有效用户ID设置成文件所有者的UID，或者将进程的有效组ID设置称为文件所有组的GID，通过`exec`实现。
5. 进程组ID。每一个进程都属于一个进程组。一个进程只能为自己或者它的子进程设置进程组ID，在它的子进程调用了exec之后，它就不能再更改子进程的进程组ID了。
6. 会话ID。一个session可以有多个进程组，一个session可以有一个控制控制（终端设备或者伪终端设备）。如果进程调用setsid()创建一个新的session，那么这个进程没有控制终端。

## 访问进程的各种ID
``` c++
pid_t getpid(void);
pid_t getppid(void);

uid_t getuid(void);
uid_t geteuid(void);

gid_t getgid(void);
gid_t getegid(void);
```

## 修改进程相关的ID
### 同时设置实际，有效和保存的设置ID
1. 使用`setxid`设置进程的实际U(G)ID，有效U(G)ID，保存的设置U(G)ID：``` c++
int setuid(uid_t uid);
int setgid(gid_t gid);
```
关于修改的规则，有三条：
    1. 如果是root用户，可以将三个ID都设置为u(g)id。
    2. 如果不是root用户，但是uid和实际用户id一样，可以将有效uid改为uid，其他不变。
    3. 否则，出错。
2. 使用此外，调用`exec`也可能会更改有效U(G)ID和保存的设置U(G)ID。无论是否设置set-user(group)-id，实际U(G)ID都不变；当设置U(G)ID位打开时，将有效U(G)ID设置为程序文件的UID，否则有效U(G)ID不变，保存的set-user(group)-id都是从有效U(G)ID复制。

总结得到以下的表格：
|ID| |`exec`| |`setuid`|
|:--:|:--:|:--:|:--:|:--:|
||set-user-ID位关闭|set-user-ID位开启| 超级用户 |  非超级用户|
|real UID|不变|不变|设置为uid|不变|
|effective UID|不变|程序文件的UID|设置为uid|设置为uid|
|saved set-UID|从effective ID复制|从effective ID复制|设置为uid|不变|


### 设置有效ID
设置进程的有效U(G)ID。如果是普通用户，可以将有效U(G)ID设置为实际U(G)ID或者保存的设置U(G)ID，如果是特权用户，可以将有效U(G)ID设置为u(g)id：``` c++
int seteuid(uid_t uid);
int setegid(gid_t gid);
```


### 交换实际ID和有效ID
交换进程的实际U(G)ID和有效U(G)ID。对于普通用户，可以交换实际U(G)ID和有效U(G)ID；还可以允许普通用户将有效U(G)ID设置为保存的设置U(G)ID。
某个参数设置为1，表示相应的ID不变。``` c++
int setreuid(uid_t ruid, uid_t euid);
int setregid(uid_t rgid, uid_t egid);
```

## 参考文献
1.《APUE》
