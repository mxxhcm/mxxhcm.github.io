---
title: python slice
date: 2019-07-13 10:30:16
tags:
 - python
 - slice
categories: python
---


## python slice index
 +---+---+---+---+---+---+
 | P | y | t | h | o | n |
 +---+---+---+---+---+---+
 0   1   2   3   4   5   6
-6  -5  -4  -3  -2  -1

## start和stop
``` txt
a[start:stop]   # 从start到stop-1的所有items
a[start:]   # 从start到array结尾的所有items
a[:stop]   # 从开始到stop-1的所有items
a[:]   # 整个array的copy
```

## step
``` txt
a[start:stop:step]   # 从start，每次加上step，不超过stop，step默认为1
```

## 负的start和stop
``` txt
a[-1]   # array的最后一个item
a[-2:]   # array的最后两个items
a[:-2]   # 从开始到倒数第三个的所有items
```

## 负的step
``` txt
a[::-1]   # 所有元素，逆序
a[1::-1]    # 前两个元素，逆序
a[:-3:-1]   # 后两个元素，逆序
a[-3::-1]   # 除了最后两个元素，逆序
```
这里加一些自己的理解，其实就是倒着数而已，包含第一个:前面的，不包含两个:之间的。


## 参考文献
1.https://stackoverflow.com/a/509297/8939281
2.https://stackoverflow.com/a/509295/8939281
