---
title: python iteration-iterable and iterator
date: 2019-10-12 15:51:26
tags:
 - python
categories: python
---

## Iteration
Iteration并不是一个具体的东西，它是一个抽象的名词，指的是一个接一个的取某个对象的每一个项。包含隐式的，显式的loop，即while，do, for等，这叫iteration。

## Iterable和iterator
而在python中，有iterator和iterable。
一个iterable object是实现了__iter__方法的object或者定义了__getitem__方法。一个iteratable object是一个可以得到iterator的object，但是它自己并不一定是iterator object。
而iterator是一个实现了__next__和__iter__方法的object。
**iterable object不一定是iterator，iterator一定是iterable object。**
**可以使用for循环的都是ieterable object，比如str，list，但是它们不是itertor，可以使用iter()方法得到iterator**
**可以next()的都是iterator**

## iter和\_\_iter\_\_
所有实现了__iter__方法的object，都是iterable object，可以通过iter()方法产生iterator object。
具体示例
```
from collections import Iterator
from collections import Iterable

class Fibs:
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def __iter__(self):
        a = self.a
        b = self.b
        while True:
            yield a
            a, b = b, a + b

real_fibs = Fibs(0,1)

print("real_fibs is iterator? ", isinstance(real_fibs, Iterator))
print("real_fibs is iterable? ", isinstance(real_fibs, Iterable))
print("iter(real_fibs) is iterator? ", isinstance(iter(real_fibs), Iterator))
print("iter(real_fibs) is iterable? ", isinstance(iter(real_fibs), Iterable))


for idx, i in enumerate(real_fibs):
    print(i)
    if idx > 10:
        break
```
其中出现了yield关键字。yield关键字的作用是每次迭代执行到该行代码时，就返回一个值，并且记住相应的位置，在下次迭代时继续从该行位置开始执行。

## next和\_\_next\_\_
代码示例
``` python
from collections import Iterator

class A:
    def __init__(self, max_v=5):
        self.max_v = max_v
        self.v = 0

    def __iter__(self):
        return self

    def __next__(self):
        # if self.v <= self.nax_v
        self.v += 1
        return self.v


if __name__ == "__main__":
    a = A()
    for idx, v in enumerate(a):
        print(idx, v)
        if (idx >= 10):
            break

```

## 参考文献
1.https://stackoverflow.com/questions/9884132/what-exactly-are-iterator-iterable-and-iteration
2.https://stackoverflow.com/a/46411740/8939281
3.https://www.jianshu.com/p/f9b547874a14
4.https://www.jianshu.com/p/1b0686bc166d
