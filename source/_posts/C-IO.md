---
title: C++ IO
date: 2019-11-05 16:00:09
tags:
 - C/C++
 - I/O
categories: C/C++
---

## I/O类
标准库提供了三类IO操作，它们分别是读写流的iostream，读写文件的fstream，读写内存中string的sstream。如下表所示：
![io](io.png)
ifstream和istringstrem都继承自istream，ofstream和ostringstream都继承自ostream。像使用cin和cout那样使用它们就行。

### IO对象的特性
1. 不能拷贝或者对IO对象赋值。所以不能将形参和返回类型设置为流类型，必须将它们设置为流引用类型。
2. 读写一个IO对象会改变它的状态，因此传递和返回的引用不能是const的。

### IO流的状态
IO操作很容易出错，一些错误是可恢复的，另一些是不可恢复的。下面是IO类中定义的函数和表示，可以帮助我们访问和操纵流的状态。
![condition](condition.png)
`strm::iostate`中存放了当前IO流的状态，这个类型是一个位集合，IO类定义了四个iostate类型的常量表达式表达特定的位类型，可以使用位运算与设置或者检测多个标志位：
- strm::badbit，表示系统级的错误
- strm::failbit，可恢复错误（到达文件结束也会置位strm::failbit，发生系统级错误时也会被置位）
- strm::eofbit，到达文件结束
- strm::goodbit，流处于未出错状态

它们用来表示流的状态，可以用`good()`, `fail()`，`eof()`, `bad()`分别查询对应标志位的状态。我们将流当做条件使用的代码其实就是使用的是状态位的状态。
可以使用`rdstate`函数获得当前流的状态，使用`setstate`对给定条件位置位，使用`clear`可以清除所错误标志位，也可以清除指定错误标志位。

### 管理缓冲区
关于缓冲区的内容，简单来说，缓冲区的作用就是减少系统级IO，提高读写效率。具体介绍可以查看[]()。
这里介绍一下导致C++中缓冲区刷新的原因：
1. 程序结束，作为main函数return的一部分，冲洗缓冲区
2. 缓冲区满时，冲洗缓冲区
3. 使用操作符endl，会在输出内容的末尾加一个`'\n'`，然后刷新缓冲区；使用flush刷新缓冲区，不附加任何字符；使用ends在输出内容的末尾加一个空字节，并不会刷新缓冲区（C++ Primer第五版上写错了）。
4. 使用`cout << unitbuf`设置为每次输出操作后都刷新缓冲区（即使不适用endl等操作符），即无缓冲，使用`cout << nounitbuf`恢复。
5. 一个输出流可能关联到另一个流，当读写关联的流时，关联到的流的缓冲区都会被刷新。


## `iostream`
C++标准IO库`iostream`提供了输入流`istream`和输出流`ostream`，一个流就是一个字符序列，从IO设备中读出或者写入IO设备。

### 标准输入输出对象
- `cin`，标准输入
- `cout`，标准输出
- `cerr`，标准错误
- `clog`，用来输出一些普通信息。

通常系统会将程序所运行的窗口和标准IO对象关联起来，读取`cin`，从当前程序关联的窗口进行读取，向`cout`,`cerr`和`clog`写入数据时，会写到同一个窗口中，可对它们进行重定向。

### 输入输出运算符
- `<<`输出运算符，接收两个运算符，左侧需要是`ostream`对象，右侧需要是要打印的对象。返回左侧运算对象，即写入给定值的`ostream`对象。
- `>>`输入运算符，接收两个运算符，左侧需要是`istream`对象，右侧从istream中读入的数据要写入的对象。返回左侧运算对象，即给定的`istream`对象。

一直有个问题就是为什么`<<`是输出，`>>`是输入，可以简单的把箭头方向当做数据流向，输出的数据流向`ostream`中的标准输出`cin`，输入时，数据从`istream`的标准输入`cin`流向变量。

### 操纵符
`endl`是一种操纵符。当把它写入到`ostream`的时候，有两个作用：
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

## `fstream`
头文件`fstream`定义了三个类型支持文件IO：
- `ifstream`从一个给定的文件读取数据；
- `ofstream`向一个给定的文件写入数据；
- `fstream`读写给定文件。

这些类型和我们之前使用的`cin`和`cout`一样，可以使用`getline`从一个`ifstream`中读取数据。`fstream`具有以下的一些特殊操作：
![fstream](fstream.png)
这些操作只有`fstream`,`ofstream`和`ifstream`对象能调用，其他类型不行。


### 使用`fstream`,`ifstream`和`ofstream`
我们想要读文件的时候，可以定义一个`fstream`对象，然后将这个对象和文件关联起来，每个`fstream`类都定义了一个名字为`open`的成员函数，它完成了一些系统相关的操作，定位给定的文件，打开为读或写模式。
在创建`fstream`对象时，我们可以提供一个文件名，此时`open`函数会被自动调用。如果定义了一个空`fstream`对象，可以手动调用`open`将它和一个文件关联起来。调用`open`可能失败，进行`open`是否成功的检测通常是一个好习惯。
`close`函数可以关闭`fstream`当前关联的文件，当一个`fstream`对象被销毁时，会自动调用`close`函数。

``` c++
// 1.构造一个ifstream并打开给定文件，文件名可以是string，也可以是C风格字符串
ifstream in(ifile);

// 2.输出文件流并未关联到任何文件。
ofstream out;
out.open(ifile+".out");

```

可以使用`fstream`代替`iostream`，因为`fstream`是`iostream`的子类。

### file mode
C++ 中的file mode如下所示：
![file_mode](file_mode.png)
具体的使用可以等用到的时候查资料。

## `sstream`
头文件`sstream`定义了三个类型支持内存IO：
- `istringstream`从`string`读取数据；
- `ostringfstream`向`string`写入数据；
- `stringstream`读写`string`。

`stringstream`特有的操作如下：
![stringstream](stringstream.png)

### 使用`istringstream`和使用`ostringstream`
最好的就是写一个例子。


## 参考文献
1.《C++ Primer第五版》
