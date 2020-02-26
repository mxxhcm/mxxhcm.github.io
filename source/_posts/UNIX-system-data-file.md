---
title: UNIX system data file
date: 2019-11-24 14:17:20
tags:
 - UNIX
categories: UNIX
---


## 系统文件
|文件名|结构|查看结构内容|头文件|查询函数|其他|
|:--:|:--:|:--:|:--:|:--:|:--:|
|/etc/passwd|passwd|man 5 passwd|pwd.h|getpwnam, getpwuid|可以使用vipw直接修改。|
|/etc/shadow|shadow|man 5 shadow|shadow.h|getspnam| |
|/etc/group|group|man 5 group|grp.h|getgrnam, getgrgid| |
|/etc/hosts|hostent|man 5 hosts|netdb.h|getnameinfo, getaddrinfo| |
|/etc/networks|netent|man 5 networks|netdb.h| getnetbyname, getnetbyaddr||
|/etc/protocols|protoent|man 5 protocols|netdb.h|getprotobyname, getprotobynumber||
|/etc/services|servent|man 5 services|netdb.h|getservbyname, getservbyprot||
|/var/run/utmp/,/var/log/wtmp/|utmp|man 5 utmp|utmp.h|getutid, getutline| |

## 其他操作
1. 查看整个口令文件：`struct pwd *getpwdent(void);`
`void setpwent(void);`
`void endpwent(void);`
2. 查看整个shadow文件:`struct pwd* getspent(void);`
`void setspent(void);`
`void endspent(void);`
3. 查看整个group文件：`struct pwd* getgrent(void);`
`void setgrent(void);`
`void endgrent(void);`

一般来说，对于第一节中列出的所有文件，都存在三个函数：`get`，`set`和`end`，它们的功能类型。
第一次调用`get`函数，返回第一项，接下来顺序的返回文件中的每一项。
`set`函数定位到数据库的开始位置。
`end`函数关闭相应的数据库。

## 附属组ID
使用newgrp(1)加GID在当前seeion内更改组ID，不加参数更改回原来的组(/etc/passwd)中的组。
附属组ID的引入可以使得一个用户至多拥有16个另外的组（常用值是16）。
有三个函数可以操作附属组ID：``` c++
// 获得当前用户的至多size个gid，如果size=0，返回总共的附属组ID的个数，如果size!=0，返回实际写入数组中的组ID的个数。
int getgroups(int size, gid_t list[]);
// 为调用进程设置附属组ID。
int setgroups(size_t size, const gid_t *list);
// 为user初始化GID。
int initgroups(const char *user, gid_t group);
```

## 登录账户记录
utmp文件记录当前登录到系统的各个用户。
wtmp文件记录各个登录和注销事件。
同样，可以使用`struct utmp* getutent(void);`
`void setutent(void);`
`void endtuent(void);`
这三个访问。
`who(1)`读取utmp文件。
`last(1)`读取wtmp文件。

## 系统标识
使用utsname(1)获得主机和操作系统相关的消息，通过定义在`<sys/utsname.h>`中的uname实现。
使用hostname(1)获取和设置主机名，通过定义在`<unistd.h>`中的gethostname和sethostname实现。

## 参考文献
1.《APUE》第三版

