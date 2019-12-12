---
title: C-style string
date: 2019-12-10 19:13:30
tags:
 - C/C++
categories: C/C++
---


## C语言中的字符串
C语言中字符串不是一种类型，而是一种约定俗成的写法。按照约定，C风格字符串存放在数组中，并且以空字符`'\0'`结束。头文件`<string.h>`提供了操作C风格字符串的一些函数。

## 字符数组
1. 使用列表初始化。```c
char str2[] = {'h', 'e', 'l', 'l', 'o'};
char str3[] = {'h', 'e', 'l', 'l', 'o', '\0'};
```
2. 使用字符串字面值初始化。``` c
char str1[] = "helloworld";
//编译器会隐式的在最后加一个"\0"，sizeof(str)会计算这个"\0", strlen(str)不会
```
这种方式是字符数组初始化的简便写法。
3. 指针和C风格字符串```c
char *messages= "hello world";
```

对于以上三种方式来说，方式1和2是等价的，这种方式中的"helloworld"存放在栈中。而第三种有些特殊，在第三种方式中，"hello world"是一个字符串字面值常量，存放在数据区的字符串常量部分[6,7,8,9]。事实上，它是一个常量字符数组，是一个不可修改的左值[9]，把它作为右值时，会进行类型转换将左值转换成右值，即使用常量字符数组首字符的地址进行初始化。


## C风格字符串操作函数
C语言标准库`<string.h>`或者C++版本的`<string.h>`提供了以下的字符串操作函数，它们的参数必须是指向以空字符结束的字符数组的指针。在函数内存不会验证这些字符串参数是否满足要求。
- `strlen(p)`，返回p指向的字符串的长度，不包括空字符
- `strcmp(p1, p2)`，p1==p2，返回0,p1>p2，返回正值，否则返回负值。
- `strcat(p1, p2)`，p2拼接到p1，返回p1
- `sctcpy(p1, p2)`，p2拷贝到p1，返回p1

有一点需要注意的是，`p2`必须能够容纳下拼接后或者拷贝后的字符串，编译器不会进行检查，这需要由程序员自己进行检查。

## C风格字符串的比较
两个C风格字符串的比较，其实比较的是指针而不是字符串本身。如下所示：```
char str1[] = "hello";
char str2[] = "world";
if(str1 < str2) //这行代码比较的不是两个字符串，而是两个指针。
```

## `string`和C风格字符串的相互转换
1. 允许使用以空字符结束的字符数组初始化`string`对象或者为`string`对象赋值。
2. `string`对象的加法运算中允许使用空字符结束的字符数组作为其运算对象，不能两个都是。
3. `string`对象的复合赋值运算中允许使用以空字符结束的字符数组作为其右侧运算对象。

#### `string`转换成C风格字符串
```
string s("hello world!");
const char *str = s.c_str();
```


## 字符串分割

## 去掉多余的空白字符

## 参考文献
1.https://stackoverflow.com/a/34957656/8939281
2.http://source-code-share.blogspot.com/2014/07/implementation-of-java-stringsplit.html
3.https://stackoverflow.com/questions/9210528/split-string-with-delimiters-in-c
4. https://stackoverflow.com/questions/17770202/remove-extra-whitespace-from-a-string-in-c

