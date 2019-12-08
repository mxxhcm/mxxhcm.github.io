---
title: UNIX struct
date: 2019-11-25 17:34:51
tags:
 - UNIX
categories: UNIX
---


struct FILE
{

};

struct DIR
{

};

## 结构体
POSIX定义了
struct dirent
{

};

struct pwd
{

};

struct spwd
{

};

struct grp
{

};

struct timespec
{

};

struct sm{

};

## 进程资源限制
```
struct rlimit{
    rlim_t rlim_cur;
    rlim_t rlim_max;

};
```

## 参考文献
1.《C++ Primer》第五版
2.https://en.cppreference.com/w/c/chrono/timespec
3.https://www.gnu.org/software/libc/manual/html_node/Elapsed-Time.html

