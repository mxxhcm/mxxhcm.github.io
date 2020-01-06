---
title: 'C char*, char [], char** and C++ string'
date: 2019-11-05 12:21:22
tags:
 - C/C++
 - C风格字符串
 - string
 - 数组
 - 指针
categories: C/C++
---

## `char arr[]`（字符数组，C类型字符串）
1. C语言中的**字符串**的概念：以NULL字节结尾的零个或者多个字符。而字符数组可以不以`'\0'`结束，而且不能为空。
2. 字符串通常存在字符数组中，这也是C语言中没有显式的字符串类型的原因。
3. 因为字符串以NULL结束，所以字符串内部不能有NULL字节。
4. 为什么选择NULL作为字符串的终止符，因为它不是一个可打印的字符。

如下所示，是`char str[]`，即字符数组的定义。`str`是自动变量，存放在栈里面。
```c
char str1[] = "hello";
char str2[] = {'h', 'e', 'l', 'l', 'o', '\0'};
char str3[] = {'h', 'e', 'l', 'l', 'o'};
```

## `char*`（字符指针）
1. `char*`是一个指针，指向一个`char`，理论上来说，它并不是一个数组。
2. `char *ptr;`并不为它指向的内容分配内存，而是只分配一个`char *`大小的内存存放指针变量`ptr`。
3. `char arr[10];`是一个数组，不是一个指针，`char *`和`char[10]`不是一个类型。

## `char**`（指向字符指针的指针）
`char**`是一个指针，指向一个指向`char`的指针。这个东西我刚开没有理解它有什么有，后来想明白了，它可以指向一个指针数组。如下所示：``` c
char *arr[] =  {
    "hello",
    "world",
    "hi"
};
char **p = arr;
```
p是字符指针的指针，指向数组中的第一个字符串，(*p)就是一个字符指针，指向第一个字符串的第一个字符，**p就是第一个字符串的第一个字符。
而`p+1`指向数组中的第二个字符串。
`p+2`指向数组中的第三个字符串。
事实上，这个p和arr的作用是一样的。`char**`经常在`main`函数中用到。

## `std::string`
字符串字面值和`string`是不同的类型，字符串字面值是为了和C语言兼容，它不是标准库中`string`对象的内容。

## 字符和字符串字面值常量
**字符串字面值常量**是用一对双引号包围一串字符。如`"hello"`, `"hi\n"`, `""`等，`'a'`是字面值字符常量。
程序在使用字符串常量时，编译器会将字符串常量存放在数据区的常量区。当一个字符串常量出现在在一个表达式中，表达式使用的值是字符串常量在内存中的地址，而不是这些字符串常量本身。可以把字符串赋值给“指向字符的指针”，即让指针指向字符串常量在内存中的地址。但是不能把字符串常量赋值给一个字符数组，即不能把字符串常量的地址赋值给字符数组。比如```c
char *message1 = "Hello world!";
```
这个代码是将字符串常量中第一个字符的地址传递给message。

## 字符数组和字面值常量的区别
```c
char message1[] = "Hello world!;
char *message2 = "Hello world!";
```
第一种方式，其实是一种约定，它等于`char message1[] = {'H', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd', '\0'};`。
而第二种方式中，"Hello world"是一个字面值常量，它在内存中只能以数组的形式存在，是一个不可修改的左值表达式。而message2实际上指向了这个字符串数组的首字符。


## 常见的指针数组
1. `main`函数的形参`char *argv[]`就是一个指针数组
2. `getline`的第一个参数是`char **lineptr`。

怎么理解？如下图所示：
![pointer_array](pointer_array.jpg)


## 参考文献
1.《C++ Primer第五版》
2.https://www.zhihu.com/question/307261590/answer/563448215
3.https://www.zhihu.com/question/307261590/answer/563630017
4.https://stackoverflow.com/questions/10004511/why-are-string-literals-l-value-while-all-other-literals-are-r-value