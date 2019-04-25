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
``` python
import torch.nn as nn


model = nn.Sequential(nn.Linear(5, 3), nn.Sequential(nn.Linear(3, 2)))

for module in model.modules():
    print(module)
```
输出如下：
> Sequential(
  (0): Linear(in_features=5, out_features=3, bias=True)
  (1): Sequential(
    (0): Linear(in_features=3, out_features=2, bias=True)
  )
)
Linear(in_features=5, out_features=3, bias=True)
Sequential(
  (0): Linear(in_features=3, out_features=2, bias=True)
)
Linear(in_features=3, out_features=2, bias=True)

可以看出来，上面总共含有四个modules。


## Module.children()
而children()不会返回它自己。
如下代码所示：
``` python
import torch.nn as nn


model = nn.Sequential(nn.Linear(5, 3), nn.Sequential(nn.Linear(3, 2)))

for child in model.children():
    print(child)
```
输出如下：
> Linear(in_features=5, out_features=3, bias=True)
Sequential(
  (0): Linear(in_features=3, out_features=2, bias=True)
)

可以看出来，上面只给出了Sequential里面的modules。


## 参考文献
1.https://discuss.pytorch.org/t/module-children-vs-module-modules/4551/2
