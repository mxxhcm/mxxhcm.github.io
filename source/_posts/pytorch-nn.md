---
title: pytorch nn
date: 2019-05-08 21:55:18
tags:
 - pytorch
 - python
categories: pytorch
---

## torch.nn
### Container

#### Module
Module是所有模型的基类。
它有以下几个常用的函数和常见的属性。
##### 常见的函数
- Module.forward() # 前向传播
- Module.modules()  # 返回module中所有的module，包含整个module，详情可见[module.modules()](htts)
- Module.named_modules() # 同时返回module的名字
- Module.children() # 返回module中所有的子module，不包含整个module，详情可见[module.modules()](htts)
- Module.named_children() # 同时返回子module的名字
- Module.parameters() # 返回模型的参数
- Module.named_parameters() # 同时返回parameter的名字
- Module.state_dict()  # 保存模型参数
- Module.load_state_dict()  # 加载模型参数
- Module.to() # 
- Module.cuda()
- Module.cpu() 
- Module.apply(fn) # 对模型中的每一个module都调用fn函数
- Module._apply()

##### 常见的属性 
- self._backend = thnn_backend
- self._parameters = OrderedDict()
- self._buffers = OrderedDict()
- self._backward_hooks = OrderedDict()
- self._forward_hooks = OrderedDict()
- self._forward_pre_hooks = OrderedDict()
- self._state_dict_hooks = OrderedDict()
- self._load_state_dict_pre_hooks = OrderedDict()
- self._modules = OrderedDict()
- self.training = True

##### 代码
关于module,children_modules,parameters的[代码]()

#### Sequential

### Convolution Layers
#### torch.nn.Conv2d
##### 类声明
应用2维卷积到输入信号中。
torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)
> Applies a 2D convolution over an input signal composed of several input planes.

##### 参数声明
in_channels (int) – 输入图像的通道
out_channels (int) – 卷积产生的输出通道数（也就是有几个kernel）
kernel_size (int or tuple) – kernel的大小 
stride (int or tuple, optional) – 卷积的步长，默认为$1$
padding (int or tuple, optional) – 向输入数据的各边添加Zero-padding的数量，默认为$0$
dilation (int or tuple, optional) – Spacing between kernel elements. Default: 1
groups (int, optional) – Number of blocked connections from input channels to output channels. Default: 1
bias (bool, optional) – If True, adds a learnable bias to the output

##### 示例
###### 例子1
用$6$个$5\times 5$的filter处理维度为$32\times 32\times 1$的图像。
``` python
import torch

model = torch.nn.Conv2d(1, 6, 5)

input = torch.randn(16, 1, 32, 32)
output = model(input)
print(output.size())
```
输出是：
> torch.Size([16, 6, 28, 28])

###### 例子2
stride和padding
``` python
import torch
import torch.nn as nn
from torch.autograd import Variable

inputs = Variable(torch.randn(64,3,32,32))

m1 = nn.Conv2d(3,16,3)
print(m1)
output1 = m1(inputs)
print(output1.size())

m2 = nn.Conv2d(3, 16, 3, padding=1)
print(m2)
output2 = m2(inputs)
print(output2.size())
```
输出
> Conv2d(3, 16, kernel_size=(3, 3), stride=(1, 1))
torch.Size([64, 16, 30, 30])
Conv2d(3, 16, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
torch.Size([64, 16, 32, 32])

### Pooling Layers
#### MaxPool2dd
MaxPool2d这个layer stride默认是和kernel_size相同的
```
import torch
from torch import nn
from torch.autograd import Variable

# maxpool2d
# class torch.nn.MaxPool2d(kernel_size,stride=None,padding=0,dilation=1,return_indices=False,ceil_mode=False)

print("input:")
input = Variable(torch.randn(30,20,32,32))
print(input.size())

m2 = nn.MaxPool2d(5)
print(m2)

for param in m2.parameters():
  print(param)

print(m2.state_dict().keys())

output = m2(input)
print("output:")
print(output.size())
```
输出
> input:
torch.Size([30, 20, 32, 32])
MaxPool2d (size=(5, 5), stride=(5, 5), dilation=(1, 1))
[]
output:
torch.Size([30, 20, 6, 6])

### Padding Layers
### Linear layers
### Dropout layers
#### Drop2D
``` python
import torch
import torch.nn as nn

m = nn.Dropout2d(0.3)
print(m)
inputs = torch.randn(1,28,28)
outputs = m(inputs)
print(outputs)
```
输出：
> Dropout2d(p=0.3)
([[[ 0.8535,  1.0314,  2.7904,  1.2136,  2.7561, -2.0429,  0.0772,
     -1.9372, -0.0864, -1.4132, -0.1648,  0.2403,  0.5727,  0.8102,
      0.4544,  0.1414,  0.1547, -0.9266, -0.6033,  0.5813, -1.3541,
     -0.0536,  0.9574,  0.0554,  0.8368,  0.7633, -0.3377, -1.4293],
    [ 0.0000,  0.0000, -0.0000, -0.0000,  0.0000, -0.0000,  0.0000,
      0.0000,  0.0000, -0.0000, -0.0000,  0.0000, -0.0000, -0.0000,
      0.0000, -0.0000,  0.0000, -0.0000,  0.0000,  0.0000, -0.0000,
     -0.0000,  0.0000,  0.0000, -0.0000, -0.0000, -0.0000, -0.0000],
      ...
    [ 0.6452, -0.6455,  0.2370,  0.1088, -0.5421, -0.5120, -2.2915,
      0.2061,  1.6384,  2.2276,  2.4022,  0.2033,  0.6984,  0.1254,
      1.1627,  1.0699, -2.1868,  1.1293, -0.7030,  0.0454, -1.5428,
      -2.4052, -0.3204, -1.5984,  0.1282,  0.2127, -2.3506, -2.2395]]])

会发现输出的数组中有很多被置为$0$了。


### Loss function


