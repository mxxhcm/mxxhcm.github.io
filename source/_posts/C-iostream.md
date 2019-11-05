---
title: C++ iostream
date: 2019-11-05 16:00:09
tags:
 - C/C++
 - iostream
categories: C/C++
---

## iostream简介
C++标准IO库`iostream`提供了输入流`istream`和输出流`ostream`，一个流就是一个字符序列，从IO设备中读出或者写入IO设备。

### 标准输入输出对象
- `cin`，标准输入
- `cout`，标准输出
- `cerr`，标准错误
- `clog`，用来输出一些普通信息。

通常系统会将程序所运行的窗口和标准IO对象关联起来，读取`cin`，从当前程序关联的窗口进行读取，向`cout`,`cerr`和`clog`写入数据时，会写到同一个窗口中。当然可以重定向。

### 输入输出运算符
- `<<`输出运算符，接收两个运算符，左侧需要是`ostream`对象，右侧需要是要打印的对象。返回左侧运算对象，即写入给定值的`ostream`对象。
- `>>`输入运算符，接收两个运算符，左侧需要是`istream`对象，右侧从istream中读入的数据要写入的对象。返回左侧运算对象，即给定的`istream`对象。

一直有个问题就是为什么`<<`是输出，`>>`是输入，可以简单的把箭头方向当做数据流向，输出的数据流向`ostream`中的标准输出`cin`，输入时，数据从`istream`的标准输入`cin`流向变量。

### 操纵符
`endl`称为操纵符。当把它写入到`ostream`的时候，有两个作用：
1. 结束当前行
2. 将与当前输出设备相关的缓冲区中的内容刷新到输出设备中。这个刷新操作可以保证目前为止程序的所有输出都真正写入输出流而不是在内存中的等待写入输出流。

### 命名空间
命名空间可以解决名自定义冲突问题。比如有两个不同的库中实现了一个同名的函数，可以通过加上命名空间进行区分。`cin`,`cout`和`endl`都定义在`std`命名空间中。在访问时需要使用以下方式：`std::cin`，`std::cout`，`std::endl`。
C++标准库定义的名字在都在`std`中。

### 读取任意的输入数据
下面的示例代码对任意的输入数据进行求和。
```
#include<iostream>

using std::cin;
using std::endl;
using std::cout;

int main()
{
    int sum = 0, value = 0;
    while(std::cin >> value)
    {
        sum += value;
    }

    std::cout << "Sum is: " << sum << std::endl;

    return 0;
}
```


## 参考文献
1.《C++ Primer第五版》
