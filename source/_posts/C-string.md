---
title: C++ string
date: 2019-11-06 16:55:51
tags:
 - string
 - C/C++
categories: C/C++
---

## `string`标准库
`string`是标准库的一部分，提供了可变长的字符串，使用时需要包含`string头文件。``` c
#include <string>
using std::string;
```

## `string`对象的初始化
1. 默认初始化，就是简单的声明。```c
string s1;
```
2. 拷贝初始化，使用等号就是拷贝初始化
```c
string s2(s1);
string s3("value");
string s4(10, 'c');
```
3. 直接初始化，不使用等号的初始化。
```c
string s2 = s1;
string s3 = "value";
```

当初始值只有一个的时候，使用直接初始化或者拷贝初始化都行，但是当初始化用到的值有多个时，只能使用直接初始化的方式。

## `string`对象的操作
1. 读写
```c
string s;
cin >> s;   //从遇到的第一个非空白符开始读入，遇到空白符就停止，且不会读入这个空白符
cout << s;
getline(cin, s);    //遇到换行符停止，读入换行符，但是不会将换行符写入s，并且遇到的第一个换行符就算。
```
2. `string`的大小
`empty()`返回`string`是否为空，
`size()`返回`string`的长度。
3. `string::size_type`
`size`返回一个无符号类型`string::size_type`的值，能够容纳放下任何`string`对象的大小。
4. 比较`string`对象
如果两个不同长度的`string`，如果短的`string`和长的`string`的共同部分完全一样，短的小。
如果两个`string`对象在某些位置上的字符不一样，按照字典序排大小，字典序在前的小。
5. `string`赋值
直接赋值字符串常量，
给字符串赋值字符串对象
6. `string`相加
7. 字面值和`string`对象相加
加号两边至少有一个对象是`string`对象

## `string`对象中字符的处理
标准库`cctype`提供了操作`string`中字符的函数。

### 范围`for`(range for)语句
1. 输出每个字符的值
```c
for( auto c: str)
    cout << c << endl;
```
2. 修改每个字符的值，加上引用
```c
for( auto & c: str)
    c = toupper(c);
```

### `[]`下标运算符
1. 迭代访问每个元素


## 参考文献
1.《C++ Primer第五版》
