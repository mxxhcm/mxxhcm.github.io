---
title: C++ namespace
date: 2019-11-06 19:19:25
tags:
 - namespace
 - C/C++
categories: C/C++
---

## 命名空间(namespace)
1. 命名空间可以解决名自定义冲突问题。比如有两个不同的库中实现了一个同名的函数，可以通过加上命名空间进行区分。
2. C++标准库定义的名字在都在`std` 命名空间，如`cin`,`cout`和`endl`等都在`std`命名空间中。在访问时需要使用以下方式：`std::cin`，`std::cout`，`std::endl`。
可以使用`using`声明来简化使用过程。```
#include<iostream>
using std::endl;
using std::cin;
int main()
{
    int val = 0;
    cin >> val;
    std::cout << val << std::endl;
    return 0;
}
```
3. 使用了`using`声明之后，在代码中就无须指出namespace了。
4. 每个名字都需要独立的`using`声明。
5. 头文件中不应该包含`using`声明，因为头文件的内容会拷贝到所有引用它的文件中，如果头文件里有某个`using`声明，那么每个使用了该头文件的文件都会有这个声明。可能会有冲突。

## 参考文献
1.《C++ Primer第五版》

