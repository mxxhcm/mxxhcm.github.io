---
title: pytorch nn.Conv2d vs nn.functional.conv2d
date: 2019-06-07 12:45:21
tags:
 - python
 - pytorch
categories: pytorch
---

## nn.Conv2d vs nn.functional.conv2d
- nn.functional包中是函数接口，是从输入到输出的一个变换，内部没有Variable，不能够构成一个layer；nn包中是nn.functional函数对应的类封装，nn中的类可能有Variable（如Conv2d)，也可能没有（如Dropout，Maxpooling）
- nn中的类一般是nn.Module的子类，继承了nn.Module的方法和属性。
- nn中的类需要传入参数实例化，然后用函数调用的方法调用实例化对象传入数据。而nn.functional是函数，不需要实例化可以直接调用，需要同时传入filters的weights和biases。

## 参考文献
1.https://www.zhihu.com/question/66782101/answer/579393790
2.https://www.zhihu.com/question/66782101/answer/246460048
