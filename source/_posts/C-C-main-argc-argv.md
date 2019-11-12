---
title: C/C++ main argc argv
date: 2019-11-12 00:43:21
tags:
 - C/C++
 - main
 - argc
 - argv
categories: C/C++
---

## main函数
`main`函数是C和C语言的入口，它的原型如下：
```c
int main(int argc, char* argv[]){...}
```
假设有一个名为prog的可执行文件，其中包含一个`main`函数，可以通过命令行选项向程序传递参数：
```shell
prog -d -o file data0
```
这些参数通过两个形参`argc`和`argv`传递给`main`函数。形参`argv`是一个数组，数组元素是字符串指针，即`argv`是一个C风格字符串指针数组。而`argc`表示的是字符串数组的长度。
当一个实参传递给`main`函数之后，`argv`的第一个元素指向程序的名字或者一个空字符串，接下来的元素依次存放命令行提供的实参。最后一个指针指向的元素应该保证为0。
注意，当使用`argv`中的实参时，注意`argv[0]`保存的是程序名字或者空字符串，可选的实参从`argv[1]`开始。

## 参考文献
1.《C++ Primer第五版》
