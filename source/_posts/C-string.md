---
title: C++ string
date: 2019-11-06 16:55:51
tags:
 - C/C++
 - STL
categories: C/C++
---

## `string`标准库
`string`是标准库的一部分，但是它不是一个容器库，不过string和容器一样支持很多操作，这里为了统一，就把它也放在STL这里部分了。
string是可变长的字符串，使用时需要包含`<string>`头文件。``` c++
#include <string>
using std::string;
```

## `string`对象的常用操作
1. 读写```c
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
4. 比较`string`对象。如果两个不同长度的`string`，如果短的`string`和长的`string`的共同部分完全一样，短的小。
如果两个`string`对象在某些位置上的字符不一样，按照字典序排大小，字典序在前的小。
5. `string`赋值。可以给string直接赋值字符串常量，也可以给string赋值string对象
6. `string`相加。字面值和`string`对象相加，加号两边至少有一个对象是`string`对象
7. `substr`返回一个子字符串。
8. 标准库`cctype`提供了操作`string`中字符的函数。
9. `s.data()`返回字符串内存空间的地址，可以看源码。



## 构造`string`
1. 默认初始化，就是简单的声明：```c++
string s1;
```
2. 直接初始化，使用等号就是拷贝初始化：```c++
string s2(s1);
string s3("value");
string s4(10, 'c');
```
3. 拷贝初始化，不使用等号的初始化：```c++
string s2 = s1;
string s3 = "value";
```
当初始值只有一个的时候，使用直接初始化或者拷贝初始化都行，但是当初始化用到的值有多个时，只能使用直接初始化的方式。
4. 和其他顺序容器相同的构造函数。
5. 其他构造函数``` c++
string s(cp, n);
string s(s2, pos2);
string s(s2, pos2, len2);
```

## `string`的增删改
1. `string`支持顺序容器的赋值运算符，`assgin`，`insert`和`erase`。
2. 除了接收迭代器的`insert`, `erase`之之外，`string`还有接收下标的版本。
3. `string`定义了`append`和`replace`函数。`replace`相当于调用`erase`和`insert`。


## `string`的搜索
每个`string`都可以调用以下搜索函数：
- `s.find(args)`。在s中查找args第一次出现的位置。
- `s.rfind(args)`。在s中查找args最后一次出现的位置。
- `s.find_first_of(args)`。在s中查找args中任何一个字符第一次出现的位置。
- `s.find_last_of(args)`。在s中查找args中任何一个字符最后一次出现的位置。
- `s.find_first_not_of(args)`。在s中查找第一个不在args中的字符。
- `s.find_last_not_of(args)`。在s中查找最后一个不在args中的字符。

## `compare`函数
每个`string`都可以调用`compare`函数和另一个`string`或者字符数组比较大小，根据s和参数指定的字符串的大小关系，返回相应的值。有相应的重载类型可以比较部分字符串。

## `string`和其他类型的转换
### `string`和C风格字符串的转换

### 数值转换
- `to_string(val)`
- `stoi(s, p, b)`
- `stol(s, p, b)`
- `stoul(s, p, b)`
- `stoll(s, p, b)`
- `stoull(s, p, b)`
- `stof(s, p)`
- `stod(s, p)`
- `stold(s, p)`


## 参考文献
1.《C++ Primer第五版》
