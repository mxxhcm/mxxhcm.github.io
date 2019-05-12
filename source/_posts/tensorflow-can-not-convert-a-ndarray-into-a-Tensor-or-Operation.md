---
title: Can not convert a ndarray into a Tensor or Operation
date: 2019-03-27 20:56:02
tags:
 - python
 - tensorflow
 - 机器学习
 - 常见问题
categories: tensorflow
---

## 错误描述
执行tensorflow代码，报错：
> Can not convert a ndarray into a Tensor or Operation.

原因是sess.run()前后参数名重了，比如outputs = sess.run(outputs)，outputs本来是自己定义的一个op，但是sess.run(outputs)之后outputs就成了一个变量，就把定义的outputs op覆盖了。

## 参考文献
1.https://blog.csdn.net/michael__corleone/article/details/79007425
