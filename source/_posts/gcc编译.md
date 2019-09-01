---
title: gcc编译
date: 2019-08-30 23:35:20
tags:
 - 工具
 - gcc
 - g++
categories: 工具
---


## 快速入门
1. 保存c++源文件为helloworld.cpp
``` c
#include <iostream>
using namespace std;
int main(){
cout<<"Hello World"<<endl;
return 0;
}
```
2. 使用g++ 编译
>>> g++ -o hello_world hello_world.cpp
3. 执行编译后的源文件
./hello_world

## 参考文献
1.file:///home/mxxmhh/Downloads/gcc-five-minutes.pdf
