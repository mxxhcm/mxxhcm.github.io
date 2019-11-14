---
title: C/C++ program in memory
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

## Text segment （代码段）
Text Segment中存放的是编译后的二进制代码，只读，固定大小。代码段也可能会放在heap或者stack下面，当它们溢出时就会覆盖代码段。

## Data segment （数据段）
数据段分为初始化段和未初始段两部分。

### Initialized Data Segment
初始化段通常简称数据段，数据段包含程序员初始化的global变量和static变量。
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

### Uninitialized Data Segment
未初始化段通常被称为bss段，"block started by symbol"。在这个段中的数据在程序开始执行前被初始化arighmetic 0。
未初始化段从数据段的结束开始，包含所有初始化为0的global和static变量以及在源代码中没有显式初始化的global和static变量。``` c
static int i;   //static 变量
int j;      //global变量
```

### Data Segment属性
，bss段包括全局区和静态区。
常量区包括字符串常量区和常变量区。

## Heap （堆）
堆是动态分配的内存，从低地址往高地址增长，存放的是用户手动分配的内存，即malloc()和calloc()等函数生成的数据存放地址。

## Stack （栈）
栈也是动态分配的内存，从高地址向低地址增长，栈中存放的是临时数据：函数参数，返回地址，局部变量等等。栈可以用来保存函数调用的现场，在递归调用中，如果调用次数太多，就可能会有stack overflow问题。


## size
可以使用`size(1)`命令查看可执行文件的text segmen, initialized data segment(data segment)和uninitialized data segment(bss)段的大小。

## 参考文献
1.https://www.geeksforgeeks.org/memory-layout-of-c-program/
2.https://www.tenouk.com/ModuleZ.html
3.https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
4.https://en.wikipedia.org/wiki/Data_segment
5.https://www.tutorialspoint.com/memory-layout-of-c-programs
