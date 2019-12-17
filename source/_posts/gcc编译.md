---
title: linux gcc
date: 2019-08-30 23:35:20
tags:
 - linux
 - 工具
categories: linux
---


## GCC
GCC是GUN项目下的编译器驱动程序，它可以将文件形式的源代码文件翻译成一个可执行的目标文件。这个翻译过程总共包含四个阶段，执行这四个阶段的程序（预处理器cpp，编译器cc1，汇编器as，链接器ld）组成了编译器驱动程序。

## 快速入门
1. 保存c++源文件为hello_world.c
``` c
#include <stdio.h>
int main(){
    printf("hello world.\n");
    return 0;
}
```

2. 使用gcc 编译
>>> gcc -o hello_world hello_world.c
3. 执行编译后的源文件
./hello_world


## 选项
- `[-Wall]`，允许发出gcc提供的所有有用的警告。
- `[-std=standard]`，指定C标准的版本，standard取值为c99, c11, gnu89等
- `[-Og]`，指定编译器优化级别，为debug可用，
- `[-m32]`，指定编译成32位程序。
- `[-m64]`，指定编译成64位程序。


## 参考文献
1.https://legacy.gitbook.com/download/pdf/book/lexdene/gcc-five-minutes
