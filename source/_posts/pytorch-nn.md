---
title: pytorch nn
date: 2019-05-08 21:55:18
tags:
 - pytorch
 - python
categories: pytorch
mathjax: true
---

## Parameter
### 一句话介绍
torch.Tensor的子类，nn.Paramter()声明的变量被赋值给module的属性时，这个变量会自动添加到moudule的parameters list中，parameters()等函数返回的迭代器中可以访问。

### API
``` python
class Parameter(torch.Tensor)
    # data是weights,requires_grad是是否需要梯度
    def __new__(cls, data=None, requires_grad=True) 
```

### 代码示例
下面的代码实现了和nn.Conv2d同样的功能，使用nn.Parameter()将手动创建的变量设置为module的paramters。
``` python
import torch
import torch.nn as nn


class CNN(nn.Module):
    
    def __init__(self):
        super(CNN, self).__init__()
        
        self.cnn1_weight = nn.Parameter(torch.rand(16, 1, 5, 5))
        self.bias1_weight = nn.Parameter(torch.rand(16))
        
        self.cnn2_weight = nn.Parameter(torch.rand(32, 16, 5, 5))
        self.bias2_weight = nn.Parameter(torch.rand(32))
        
        self.linear1_weight = nn.Parameter(torch.rand(4 * 4 * 32, 10))
        self.bias3_weight = nn.Parameter(torch.rand(10))
        
    def forward(self, x):
        x = x.view(x.size(0), -1)
        out = F.conv2d(x, self.cnn1_weight, self.bias1_weight)
        out = F.relu(out)
        out = F.max_pool2d(out)
        
        out = F.conv2d(x, self.cnn2_weight, self.bias2_weight)
        out = F.relu(out)
        out = F.max_pool2d(out)
        
        out = F.linear(x, self.linear1_weight, self.bias3_weight)
        return out
```

## Container
### Module
Module是所有模型的基类。
它有以下几个常用的函数和常见的属性。
#### 常见的函数
- add_module(name, module)  # 添加一个子module到当前module
- apply(fn) # 对模型中的每一个submodule（调用.children()得到的）都调用fn函数
- \_apply() # in-place
- children() # 返回module中所有的子module，不包含整个module，[详情可见](https://github.com/mxxhcm/code/blob/master/pytorch/pytorch_test/torch_module_child_parameter.py)
- buffers(recurse=True)     # 返回module buffers的迭代器 
- cuda(device=None)    # 将model para和buffer转换到gpu
- cpu()     # 将model para和buffer转换到cpu
- double()  #将float的paramters和buffer转换成double
- float()
- forward(\*input) # 前向传播
- eval()    # 将module置为evaluation mode，只影响特定的traing和evaluation modules表现不同的module，比如Dropout和BatchNorm，一般Dropout在训练时使用，在测试时关闭。
- train(mode=True)  # 设置模型为train mode
- load_state_dict(state_dict, strict=True)  # 加载模型参数
- modules()  # 返回network中所有的module的迭代器，包含整个module，[详情可见](https://github.com/mxxhcm/code/blob/master/pytorch/pytorch_test/torch_module_child_parameter.py)
- named_modules() # 同时返回包含module和module名字的迭代器
- named_children() # 同时返回包含子module和子module名字的迭代器
- named_parameters() # 同时返回包含parameter和paramter名字的迭代器
- parameters() # 返回模型参数的迭代器
- state_dict(destination=None, prefix='', keep_vars=False)  # 返回整个module的state，包含parameters和buffers。
- zero_grad()   # 设置model parameters的gradients为$0$
- to(\*args, \*\*kwargs) # 移动或者改变parameters和buffer的类型或位置

#### 常见的属性 
- self.\_backend = thnn_backend
- self.\_parameters = OrderedDict()
- self.\_buffers = OrderedDict()
- self.\_backward_hooks = OrderedDict()
- self.\_forward_hooks = OrderedDict()
- self.\_forward_pre_hooks = OrderedDict()
- self.\_state_dict_hooks = OrderedDict()
- self.\_load_state_dict_pre_hooks = OrderedDict()
- self.\_modules = OrderedDict()
- self.training = True

#### 代码示例
##### apply
``` python
def init_weights(m):
    print(m)
    if type(m) == nn.Linear:
        m.weight.data.fill_(1.0)
        print(m.weight)
net = nn.Sequential(nn.Linear(2, 2), nn.Linear(2, 2))
net.apply(init_weights)
```
##### module
关于module,children_modules,parameters的[代码](https://github.com/mxxhcm/code/blob/master/pytorch/pytorch_test/torch_module_child_parameter.py)

### Sequential

## Convolution Layers
### torch.nn.Conv2d
#### API
2维卷积。
``` python
torch.nn.Conv2d(
    in_channels,    # int，输入图像的通道
    out_channels,   # int，卷积产生的输出通道数（也就是有几个kernel） 
    kernel_size,    # int or tuple – kernel的大小 
    stride=1,       # int or tuple, 可选，步长，默认为$1$
    padding=0,      # int or tuple, 可选，向各边添加Zero-padding的数量，默认为$0$
    dilation=1,     # int or tuple, 可选，Spacing between kernel elements. Default: 1
    groups=1,       # int, 可选， Number of blocked connections from input channels to output channels. Default: 1
    bias=True       # bool，可选，如果为True,给output添加一个可以学习的bias
)
```

#### 示例
##### 例子1
用$6$个$5\times 5$的filter处理维度为$32\times 32\times 1$的图像。
``` python
import torch

model = torch.nn.Conv2d(1, 6, 5)

input = torch.randn(16, 1, 32, 32)
output = model(input)
print(output.size())
# output: torch.Size([16, 6, 28, 28])
```

##### 例子2，stride和padding
``` python
import torch
import torch.nn as nn
from torch.autograd import Variable

inputs = Variable(torch.randn(64, 3, 32, 32))

m1 = nn.Conv2d(3, 16, 3)
print(m1)
# output: Conv2d(3, 16, kernel_size=(3, 3), stride=(1, 1))
output1 = m1(inputs)
print(output1.size())
# output: torch.Size([64, 16, 30, 30])

m2 = nn.Conv2d(3, 16, 3, padding=1)
print(m2)
# output: Conv2d(3, 16, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
output2 = m2(inputs)
print(output2.size())
# output: torch.Size([64, 16, 32, 32])
```

## Pooling Layers
### MaxPool2dd
#### API
``` python
class torch.nn.MaxPool2d(
    kernel_size,
    stride=None,
    padding=0,
    dilation=1,
    return_indices=False,
    ceil_mode=False
)
```

MaxPool2d默认layer stride默认是和kernel_size相同的
#### 代码示例
```
import torch
from torch import nn
from torch.autograd import Variable
# maxpool2d

input = Variable(torch.randn(30,20,32,32))
print(input.size())
# outputtorch.Size([30, 20, 32, 32])

m1 = nn.MaxPool2d(2)
output = m1(input)
print(output.size())
# output: torch.Size([30, 20, 16, 16])


m2 = nn.MaxPool2d(5)
print(m2)
# output: MaxPool2d (size=(5, 5), stride=(5, 5), dilation=(1, 1))

for param in m2.parameters():
  print(param)

print(m2.state_dict().keys())
# output: []

output = m2(input)
print(output.size())
# output: torch.Size([30, 20, 6, 6])
```

## Padding Layers

## Linear layers
### Linear
#### API
``` python
torch.nn.Linear(
    in_features, 
    out_features, 
    bias=True
)
```
#### 代码示例
``` python
m = nn.Linear(20, 30)
input = torch.randn(128, 20)
output = m(input)
print(output.size())
torch.Size([128, 30])
```

## Dropout layers
### Drop2D
#### API

#### 代码示例
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

## Loss function
