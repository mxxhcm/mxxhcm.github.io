---
title: pytorch initialize parameters
date: 2019-05-08 22:01:24
tags:
 - pytorch
 - python
categories: pytorch
---

## 神经网络参数初始化
### 方法$1$.Model.apply(fn)
[示例](https://github.com/mxxhcm/myown_code/blob/master/pytorch/tutorials/initialize/apply.py)如下
``` python
import torch.nn as nn
def init_weights(m):
  print(m)
  if type(m) == nn.Linear:
    m.weight.data.fill_(1.0)
    print(m.weight)

net = nn.Sequential(nn.Linear(2, 2), nn.Linear(2, 2))
net.apply(init_weights)
```
输出结果如下：
> Linear(in_features=2, out_features=2, bias=True)
Parameter containing:
tensor([[1., 1.],
        [1., 1.]], requires_grad=True)
Linear(in_features=2, out_features=2, bias=True)
Parameter containing:
tensor([[1., 1.],
        [1., 1.]], requires_grad=True)
Sequential(
  (0): Linear(in_features=2, out_features=2, bias=True)
  (1): Linear(in_features=2, out_features=2, bias=True)
)
Linear(in_features=2, out_features=2, bias=True)
Linear(in_features=2, out_features=2, bias=True)

其中最后两行为net对象调用self.children()函数返回的模块，就是模型中所有网络的参数。事实上，调用net.apply(fn)函数，会对self.children()中的所有模块应用fn函数，

## 参考文献
