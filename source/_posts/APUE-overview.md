---
title: APUE-overview
date: 2019-11-11 16:31:06
tags:
 - UNIX
 - APUE
categories: UNIX
---

## APUE是什么
APUE是Advandaced Programming in the UNIX environment的简称，即UNIX高级环境编程。
很多人都把这本书和UNP(UNIX Network Programming)当做UNIX编程的神书。一直想拜读这两本书，但是每次翻开之后就放弃了，因为看不懂它在讲些什么，这篇博客简单介绍了APUE到底是什么，它能用来干什么。

## APUE包含的内容
UNIX系统提供了两类程序设计接口：
1. 系统调用接口：UNIX为程序运行提供的大量服务－打开文件，读文件，启动一个新程序，分配存储区以及获得当前时间等等，这些服务被称为系统调用接口(system call interface)。
2. 标准C库：提供大量广泛用于C程序中的函数（格式化输入出入，字符串比较等等）。

APUE这本书并不是介绍UNIX中有哪些系统调用接口和库函数，这些可以从《UNIX程序员手册》中找到，APUE给出的是这些系统调用接口和库函数该怎么使用，以及它们的基本原理。

## UNIX标准
20世纪80年代，有各个版本的UNIX，为了让这些UNIX版本统一起来，人们制定了数个国际标准，包含C程序设计的ANSI C标准，IEEE POSIX标准，以及X/Open可移植性等等。

## APUE样例
APUE中给出的所有示例代码，都是ANSI C编写的。这些程序的[下载地址](http://www.apuebook.com/code2e.html)。


## 参考文献
1.《APUE》第三版
