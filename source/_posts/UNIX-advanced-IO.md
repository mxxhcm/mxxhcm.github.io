---
title: UNIX advanced IO
date: 2020-03-03 15:21:19
tags:
 - UNIX
categories: UNIX
---


## 记录锁

## select和poll

### select和pselect

### poll

## 异步IO

## readv和writev
readv从文件描述符指定的文件中读取相应的数据到多个缓冲区。
writev将多个缓冲区中的内容写入文件描述符中。


## readn和writen
适用于已经知道要读取和写入的字节数量的情景。其实就是多次调用非阻塞的read和write进行操作。


## 存储映射IO
mmap将一个磁盘文件映射到存储空间的一个缓冲区上。从缓冲区读，就相当于读文件中的相应字节。从缓冲区写，就相当于写文件中的相应字节。


## 参考文献
1.《APUE》第三版
