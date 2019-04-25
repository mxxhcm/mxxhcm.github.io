---
title: pytorch Module.children() vs Module.modules()
date: 2019-04-25 21:06:46
tags:
 - pytorch
 - python
categories: python
---

## Module.modules()
modules()会返回所有的模块，包括它自己。
如下代码所示：


## Module.children()
而children()不会返回它自己。
如下代码所示：


## 参考文献
1.https://discuss.pytorch.org/t/module-children-vs-module-modules/4551/2
