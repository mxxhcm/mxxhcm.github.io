---
title: C/C++ main argc argv
date: 2019-11-12 00:43:21
tags:
 - C/C++
categories: C/C++
---

## main函数
`main`函数是C语言和C++ 程序的入口，C和C++ 的标准要求它们的实现必须支持以下两种形式：
```c
int main(void) {...}
int main(int argc, char* argv[]){...}
```
任何C和C++ 库实现都必须实现以上两种形式的`main`，除此以外，可以根据标准进行其他扩展实现，但是这样子可能在一个平台上能运行的程序在另一个平台上不能运行，即除了标准的两种`main`，其他扩展都是不可移植的。
需要注意的一点是，C和C++ 标准对于`main`的扩展有要求，C++ 标准要求所有的`main`都必须返回`int`类型，只有它们的参数可以改变。而C要自由一些，可以有`void main(char, dobule)`的实现，但是在C++ 标准中这是不支持的。`int main(int ,char*, char**)`在C和C++ 标准中都是允许的。

## main函数示例
假设有一个名为prog的可执行文件，其中包含一个`main`函数，可以通过命令行选项向程序传递参数：
```shell
prog -d -o file data0
```
这些参数通过两个形参`argc`和`argv`传递给`main`函数。形参`argv`是一个数组，数组元素是字符串指针，即`argv`是一个C风格字符串指针数组。而`argc`表示的是字符串数组的长度。
当一个实参传递给`main`函数之后，`argv`的第一个元素指向程序的名字或者一个空字符串，接下来的元素依次存放命令行提供的实参。最后一个指针指向的元素应该保证为0。
注意，当使用`argv`中的实参时，注意`argv[0]`保存的是程序名字或者空字符串，可选的实参从`argv[1]`开始。


## 参考文献
1.《C++ Primer第五版》
2.https://stackoverflow.com/questions/9554513/c-main-vs-c-main
3.《你必须知道的495个C语言问题》
