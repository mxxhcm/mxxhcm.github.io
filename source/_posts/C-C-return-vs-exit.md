---
title: C/C++ return vs exit
date: 2019-11-12 13:51:20
tags:
 - return
 - exit
 - C/C++
categories: C/C++
---

## `return`
1. `return`是C/C++语言的关键字，是语言级别的。
2. `return`结束一个函数的执行。
3. 

## `exit`
1. `exit()`是一个函数，它是对系统调用`_exit()`的封装，是系统调用层次的。
2. `exit()`结束一个进程，删除进程使用的内存空间，并且将应用程序的一个状态返回给OS，这个状态标识了进程的运行信息。
3. `exit(0)`表示正常运行程序并退出程序；
`exit(1)`表示非正常运行导致退出程序。

## `return`和`exit`
对于我们自定的函数，可以return给操作系统，交给相关的处理程序调用exit或者程序自身直接调用exit。

### C++中的区别
在C++中，退出程序时，`exit`并不会调用局部非静态对象的析构函数，而`return`会调用局部非静态对象的析构函数。


## 参考文献
1.https://stackoverflow.com/questions/461449/return-statement-vs-exit-in-main
2.https://www.geeksforgeeks.org/return-statement-vs-exit-in-main/
3.https://www.zhihu.com/question/26591968/answer/33839473
4.https://www.zhihu.com/question/26591968/answer/33330774
