---
title: Undefined reference to pthread_create in Linux
date: 2018-12-20 20:30:34
tags: 
  - gcc
  - codeblocks
  - Linux
  - 常见问题
categories: Linux
---
## 问题

在阅读自然语言处理的一篇论文时，读到了bype pair encoding(bpe)算法。在github找到了一个实现[fastBPE](https://github.com/glample/fastBPE), 算法是用C++写的，在编译的过程中遇到了问题"Undefined reference to pthread_create in Linux", 

## terminal下解决方案
查阅资料了解到pthread不是Linux操作系统默认的库函数，所以需要在编译的时候将pthread链接该库函数，后来在看fastBPE的文档时发现文档中已经有说明:
Compile with:
``` shell
g++ -std=c++11 -pthread -O3 fast.cc -o fast
```

## codeblocks下解决方案
上面给出的方案是使用gcc在terminal进行编译时加入静态库，但是对于不习惯在命令行使用gdb进行调试的人来说没有用。
在codeblocks中，如果要链接静态库,找到Settings --> Compiler... --> Linker settings，点击add，添加相应的库函数即可。

## 参考文献
1:<https://stackoverflow.com/questions/1662909/undefined-reference-to-pthread-create-in-linux>
2:<https://blog.csdn.net/zhaoyue007101/article/details/7705753>
