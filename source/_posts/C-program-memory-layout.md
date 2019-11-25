---
title: C/C++ program memory layout
date: 2019-10-19 20:41:02
tags:
 - C/C++
 - memeory
categories: C/C++
---

## C语言程序在内存中的组成部分
C语言程序在内存中的构成如下所示：
![c_program_in_memory](c_in_memory.jpg)
即一个C语言程序在内存中由text segment, data segment，heap和stack组成。在程序执行的过程中，[`exec`]()从程序文件中读取代码段和初始化数据段相关内容，然后`exec`负责将未初始化数据(bss)段中的内容初始化为0。
所有的static和全局变量存放在数据段，所有的自动变量和临时变量都存在stack中，动态变量存放在heap中。
所有的函数参数存放在stack中，每一个函数都有一个不同的stack frames。
在栈的最上面，存放的是命令行参数和环境变量，这个空间是不可扩展的，因为它在进程地址空间的最上面所以不能向上扩展，而它下面的所有栈帧是不可移动的，所以也不能向下扩展。

## Text segment （代码段）
Text Segment，CPU执行的机器指令部分。通常，正文段是可共享的，所以即使是频繁执行的程序在存储中也只有一个副本。正文段通常是只读的，防止程序由于意外而修改指令。代码段也可能会放在heap或者stack下面，当它们溢出时就会覆盖代码段。

## Initialized Data Segment
Initialized data segment（初始化段），通常简称数据段，数据段包含程序员初始化的global变量和static变量。
这个段也可以被划分成初始只读区域和初始读写区域，例如：```c
char s[] = "hello world";   //s在初始化数据段，存放的是"hello world"
int debug = 1;      //全局初始化变量
static int i = 10;  //全局静态变量

const char *str = "hello world";    //字符串常量存放在字符串常量区，str是全局指向常量的指针，存放的是字符串常量"hello world"在内存中的地址。
int main()
{
    statit int si;  //局部static 变量。
}
```

## Uninitialized Data Segment
Uninitizlized data segment（未初始化段），通常被称为bss段，这个名称来自于早期汇编程序的一个操作符，意思是"block started by symbol"。在程序开始执行前，内核将这个段中的数据初始化0或者空指针。
未初始化段从数据段的结束开始，包含所有初始化为0的global和static变量以及在源代码中没有显式初始化的global和static变量。``` c
static int i;   //static 变量
int j;      //global变量
```

## Heap （堆）
堆是动态分配的内存，从低地址往高地址增长，存放的是用户手动分配的内存，即malloc()和calloc()等函数生成的数据存放地址。位于bss段和栈之间。

## Stack （栈）
栈也是动态分配的内存，从高地址向低地址增长，栈中存放的是临时数据：自动变量，函数参数，返回地址，局部变量等等。栈可以用来保存函数调用的现场，在递归调用中，如果调用次数太多，就可能会有stack overflow问题。


## size
可以使用`size(1)`命令查看可执行文件的text segmen, initialized data segment(data segment)和uninitialized data segment(bss)段的大小。

## 参考文献
1.https://www.geeksforgeeks.org/memory-layout-of-c-program/
2.https://www.tenouk.com/ModuleZ.html
3.https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
4.https://en.wikipedia.org/wiki/Data_segment
5.https://www.tutorialspoint.com/memory-layout-of-c-programs
