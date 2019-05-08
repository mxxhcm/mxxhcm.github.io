---
title: pytorch nn functional
date: 2019-05-08 21:55:37
tags:
 - pytorch
 - python
categories: pytorch
---

## torch.nn.functional

### convoludion functions
#### conv2d
```
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.autograd import Variable

inputs = Variable(torch.randn(64,3,32,32))

filters1 = Variable(torch.randn(16,3,3,3))
output1 = F.conv2d(inputs,filters1)
print(output1.size())

filters2 = Variable(torch.randn(16,3,3,3))
output2 = F.conv2d(inputs,filters2,padding=1)
print(output2.size())
```
输出
> torch.Size([64, 16, 30, 30])
torch.Size([64, 16, 32, 32])

### relu functions

### pooling functions
### dropout functions
#### 例子
``` python
import torch
import torch.nn.functional as F
 
x = torch.randn(1, 28, 28)
y = F.dropout(x, 0.5, True)
y = F.dropout2d(x, 0.5)
 
print(y)
```
注意$9$中说的问题，不过可能已经被改正了，注意一些就是了。

### linear functions
### loss functions


