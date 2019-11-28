---
title: C/C++ separate compilation
date: 2019-11-28 23:24:38
tags:
 - C/C++
 - 分离式编译
categories: C/C++
---


## 功能文件头文件和实现
### 头文件
``` cpp
// print_string.h
#ifndef PRINT_STRING
#define PRINT_STRING

void print_string(const char *str);

#endif
```

### 实现
``` cpp
// print_string.cpp
#include "print_string.h"
#include <stdio.h>

void print_string(const char *str)
{
    printf("%s\n",  str);
}
```

## 测试文件实现
``` cpp
#include "print_string.h"


int main()
{

    char str[] = "hello world";
    print_string(str);
    return 0;
}
```

## 编译和链接
执行以下命令
``` shell
g++ main.cpp print_string.cpp -o main
./main
```

## 理解
在`main`中包含了`print_string.h`头文件，相当于对函数进行了声明。然后使用`g++`编译的时候相当于提供了`print_string`的实现。


