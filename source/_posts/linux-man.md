---
title: ' linux man'
date: 2019-11-07 16:12:01
tags:
 - linux
categories: linux
---

## man的简介
man是linux下的一个文档查看命令，它把所有的命令分为9个部分。
1. user command
2. system calls，系统调用，内核提供的函数
3. library functions，库函数
4. specifal files，像/dev/文件夹下的
5. file formats，查看相应文件的文件格式
6. games，
7. conventions
8. administration and privileged commands，系统管理员命令
9. math library functions，数学函数库
10. tcl functions

一个命令可能会在多个类别中出现，在查找相应的命令的时候，可使用`man`加上具体的数字，就可以得到对应类中的手册。
具体示例
查看passwd和group的文件格式：
``` shell
man 5 passwd
man 5 group
```

## man命令的介绍
- 名字(NAME)，命令简介
- 格式(SYSOPSIS)，命令格式
- 描述(DESCRIPTION)，介绍命令的参数options
- 作者(AUTHOR)，命令的作者
- 提交BUG(REPORTINT BUGS)，提交一些bug的地址
- 版权(COPYRIGHT)
- 更多参考(SEE ALSO)

比如`man ls`的输出：
``` txt
NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about  the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort  is  speci‐
       fied.

       Mandatory  arguments  to  long  options are mandatory for short options
       too.

       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author
...
```

## 命令格式
拿`ls`举个例子，`man ls`的SYNOPSIS如下：
``` txt
ls [OPTION]... [FILE]...
```
其中`[]`表示可选，`...`表示有多个。`[OPTION]...`表示有多个可选参数，`[FILE]...`表示有多个可选文件。

## 命令参数
DESCRIPTION部分会给定更加详细一些的介绍，以及给出具体参数的含义。
```
-a, --all
    do not ignore entries string with .
--author
    with -l, print the author of each file
```
其中`-`是缩写，`--`是全称，他们的作用其实是一样的。


## 参考文献
1.https://linux.die.net/man/
2.https://www.cnblogs.com/shanyu20/p/10943393.html
