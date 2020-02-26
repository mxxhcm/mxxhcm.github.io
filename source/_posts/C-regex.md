---
title: C++ regex
date: 2020-02-18 18:21:06
tags:
 - C/C++
categories: C/C++
---


## 正则表达式
regex定义在regex包中。

## regex类
regex是一个正则表达式类。
输入是string，使用smatch保存匹配的相关信息，使用sregex_iterator获得某个string中所有匹配的子串。
输入是`const char *`，使用regex, cmatch，和cregex_iterator。

## C++ 正则使用流程
给定一个输入（待搜索序列）string。
创建一个regex对象，调用regex_search(是否存在一个子串和regex匹配)或者regex_match（整个字符串是否和regex匹配）进行匹配，如果匹配成功，将匹配到的内容写入一个smatch对象，并且返回一个true的布尔变量，否则返回false。
或者可以使用sregex_iterator查找所有的匹配子串，返回一个smatch对象的引用，或者一个指向smatch对象的指针。

## smatch操作


## 参考文献
1.《C++ Primer》
