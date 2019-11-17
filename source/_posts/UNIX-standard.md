---
title: UNIX standard and implement
date: 2019-11-11 18:09:41
tags:
 - UNIX
 - UNIX standard
categories: UNIX
---

## UNIX standard

### ISO C
ISO C是国际标准化组织给出的C语言的标准。它包含两部分：C语言的语法和语义，标准库。但是标准只是给出了C标准函数的原型和功能，并没有给出他们的实现。具体的实现由编译器决定，只要编译器声称它们支持ISO C标准，那么这个编译器就必须严格遵守ISO C标准中的各项规定。
根据C语言的发展来说：
1972年，丹里斯发明了C语言，这个版本的C语言叫做K&R C。
ISO C的前身是ANSI C，1989年，美国国家标准学会(ANSI)提出了ANSI标准X3.159-1989，这个标准就是ANSI C，它也被采纳为国际标准ISO/IEC 9899:1990，也就是ISO C90。
后来有陆续有了ISO C99，ISO C11等等。

按照ISO标准定义的头文件，可以将ISO C库分为24个区。下面要介绍的POSIX.1标准包含ISO C的头文件以及另外一些头文件。

### IEEE POSIX
POSIX是由国际电气和电子工程学会制定的标准族。POSIX是可移植操作系统接口（Portable Operating System Interface）。它原来只是IEEE操作系统接口的标准，后来扩展成了许多其他标准和标准草案，如shell和实用程序等。


### Single UNIX Specification

## UNIX implement
1. SVR4
2. 4.4BSD
3. FreeBSD
4. Linux
5. Mac OS X
6. Solaris
7. 其他UNIX系统

## 标准和实现的关系
## 参考文献
1.《APUE》
2.https://www.zhihu.com/question/40175738/answer/154308906
