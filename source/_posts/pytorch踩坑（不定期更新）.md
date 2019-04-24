---
title: pytorch踩坑（不定期更新）
date: 2019-03-13 15:10:22
tags:
 - pytorch
 - python
 - 机器学习
 - 深度学习
categories: python
---

## 常见问题
### RuntimeError: CUDNN_STATUS_ARCH_MISMATCH
CUDNN doesn't support CUDA arch 2.1 cards.
CUDNN requires Compute Capability 3.0, at least. 
意思是GPU的加速能力不够，CUDNN只支持CUDA Capability 3.0以上的GPU加速，实验室主机是GT620的显卡，2.1的加速能力。
GPU对应的capability: https://developer.nvidia.com/cuda-gpus 
所以，对于不能使用cudnn对cuda加速的显卡，我们可以设置cudnn加速为False，这个默认是为True的
torch.backends.cudnn.enabled=False
但是，由于显卡版本为2.1，太老了，没有二进制版本。所以，还是会报其他错误，因此，就别使用cpu进行加速啦。

### 查看cuda版本
~#:nvcc --version

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

## torch.multiprocessing
### join
等待调用join()方法的线程执行完毕，然后继续执行。
可参见github[https://github.com/mxxhcm/myown_code/tree/master/pytorch/tutorials/torch_multiprocess/mnist_hogwild](官方demo)。

### share_memory\_()
在多个线程之间共享参数，如下代码所示。可以用来实现A3C。
``` python
import torch.multiprocessing as mp
import torch
import time
import os


def proc(sec, x):
   print(os.getpid(),"  ", x)
   time.sleep(sec)
   print(os.getpid(), "  ", x)
   x += sec
   print(str(os.getpid()) + "  over.  ", x)


if __name__ == '__main__':
   num_processes = 3
   processes = []
   x = torch.ones([3,])
   x.share_memory_()
   for rank in range(num_processes):
     p = mp.Process(target=proc, args=(rank + 1, x))
     p.start() 
     processes.append(p)
   for p in processes:
     p.join()
   print(x)
```
输出结果如下所示：
> python share_memory.py 
7739    tensor([1., 1., 1.])
7738    tensor([1., 1., 1.])
7737    tensor([1., 1., 1.])
7737    tensor([1., 1., 1.])
7737  over.   tensor([2., 2., 2.])
7738    tensor([2., 2., 2.])
7738  over.   tensor([4., 4., 4.])
7739    tensor([4., 4., 4.])
7739  over.   tensor([7., 7., 7.])
tensor([7., 7., 7.])

我们可以发现$7739$这个线程中，传入的$x$还是和最开始的一样，但是在$7738$线程更新完$x$之后，$7739$使用的$x$就已经变成了更新后的$x$。所以，我猜测这里面应该是有一个对$x$的锁，保证$x$在同一时刻只能被一个线程访问。

## torch
### torch.cat
#### 函数原型
将多个tensor在某一个维度上（默认是第0维）拼接到一起（除了拼接的维度上，其他维度的shape必须一定），最后返回一个tensor。
torch.cat(tensors, dim=0, out=None) → Tensor
> Concatenates the given sequence of seq tensors in the given dimension. All tensors must either have the same shape (except in the concatenating dimension) or be empty.

#### 参数
tensors (sequence of Tensors) – 任意类型相同python序列或者tensor
dim (int, optional) - 在第几个维度上进行拼接(只有在拼接的维度上可以不同，其余维度必须相同。
out (Tensor, optional) – 输出的tensor

#### 例子
```python
import torch

x1 = torch.randn(3, 4, 4)
x2 = torch.randn(3, 1, 4)
x = torch.cat([x1, x2], 1)
print(x.size())
```
输出如下：
> torch.Size([3, 5, 4])

### img
torch中图像的shape是('RGB',width, height)，而numpy和matplotlib中都是(width, height, 'RGB')
matplotlib.pyplot.imshow()需要的参数是图像矩阵，如果矩阵中是整数，那么它的值需要在区间[0,255]之内，如果是浮点数，需要在[0,1]之间。
> Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers).


## torch.autograd(torch.autograd.Variable和torch.autograd.Function)

### Variable(class torch.autograd.Variable)
#### 声明一个tensor
torch.zeros,torch.ones,torch.rand,torch.Tensor
``` python
import torch
torch.zeros(2,3)
torch.ones(2,3)
torch.rand(2,4)
torch.randn(2,3)
torch.randn(2,3).type()
torch.Tensor(3,4)
torch.Tensor(3,4).uniform_(0,1)
torch.Tensor(3,4).normal_(0,1)
torch.Tensor(3,4).fill_(5)
torch.arange(1,3,0.4)
```

#### tensor的各种操作
``` python
import torch
a = torch.ones(2,3)
b = torch.ones(2,3)
```

##### 加操作
``` python
print(a+b)                #方法1
c = torch.add(a,b)    #方法2
torch.add(a,b,result)    #方法3
a.add(b)                    #方法4,将a加上b，且a不变
a.add_(b)                #方法5,将a加上b并将其赋值给a
```
##### 转置操作
```python
print(a.t())               # 打印出tensor a的转置
print(a.t_())                 #将tensor a 转置，并将其赋值给a
```

##### 求最大行和列
``` python
torch.max(tensor,dim)
np.max(array,dim)
```

##### 和relu功能比较类似。
``` python
torch.clamp(tensor, min, max,out=None)
np.maximun(x1, x2)  # x1 and x2 must hava the same shape
```

#### tensor和numpy转化
##### convert tensor to numpy
``` python
a = torch.ones(3,4)
b = a.numpy()
```
##### convert numpy to tensor
``` python
a =  numpy.ones(4,3)
b = torch.from_numpy(a)
```

#### Variable和Tensor
Variable
图1.Variable

##### 属性
如图1,Variable wrap a Tensor,and it has six attributes,data,grad,requies_grad,volatile,is_leaf and grad_fn.We can acess the raw tensor through .data operation, we can accumualte gradients w.r.t this Variable into .grad,.Finally , creator attribute will tell us how the Variable were created,we can acess the creator attibute by .grad_fn,if the Variable was created by the user,then the grad_fn is None,else it will show us which Function created the Variable.
if the grad_fn is None,we call them graph leaves
``` python
Variable.shape  #查看Variable的size
Variable.size()
```

##### parameters
``` python
torch.autograd.Variable(data,requires_grad=False,volatile=False)
```
requires_grad : indicate whether the backward() will ever need to be called

##### backward
backward(gradient=None,retain_graph=None,create_graph=None,retain_variables=None)
如果Variable是一个scalar output，我们不需要指定gradient，但是如果Variable不是一个scalar，而是有多个element，我们就需要根据output指定一下gradient，gradient的type可以是tensor也可以是Variable，里面的值为梯度的求值比例，例如
``` python
x = Variable(torch.Tensor([3,6,4]),requires_grad=True)
y = Variable(torch.Tensor([5,3,6]),requires_grad=True)
z = x+y
z.backward(gradient=torch.Tensor([0.1,1,10]))
```
这里[0.1,1,10]分别表示的是对正常梯度分别乘上$0.1,1,10$，然后将他们累积在leaves Variable上
``` python
detach()    #
detach_()
register_hook()
register_grad()
```

### Function(class torch.autograd.Funtion)
#### 用法
Function一般只定义一个操作，并且它无法保存参数，一般适用于激活函数，pooling等，它需要定义三个方法，__init__(),forward(),backward()（这个需要自己定义怎么求导）
Model保存了参数，适合定义一层，如线性层(Linear layer),卷积层(conv layer),也适合定义一个网络。
和Model的区别，model只需要定义__init()__,foward()方法，backward()不需要我们定义，它可以由自动求导机制计算。

Function定义只是一个函数，forward和backward都只与这个Function的输入和输出有关
#### functions
``` python
import torch
from torch.autograd import Variable

class MyReLU(torch.autograd.Function):
    """
    We can implement our own custom autograd Functions by subclassing
    torch.autograd.Function and implementing the forward and backward passes
    which operate on Tensors.
    """

    def forward(self, input):
        """
        In the forward pass we receive a Tensor containing the input and return a
        Tensor containing the output. You can cache arbitrary Tensors for use in the
        backward pass using the save_for_backward method.
        """
        self.save_for_backward(input)
        return input.clamp(min=0)

    def backward(self, grad_output):
        """
        In the backward pass we receive a Tensor containing the gradient of the loss
        with respect to the output, and we need to compute the gradient of the loss
        with respect to the input.
        """
        input, = self.saved_tensors
        grad_input = grad_output.clone()
        grad_input[input < 0] = 0
        return grad_input

dtype = torch.FloatTensor
# dtype = torch.cuda.FloatTensor # Uncomment this to run on GPU

# N is batch size; D_in is input dimension;
# H is hidden dimension; D_out is output dimension.
N, D_in, H, D_out = 64, 1000, 100, 10

# Create random Tensors to hold input and outputs, and wrap them in Variables.
x = Variable(torch.randn(N, D_in).type(dtype), requires_grad=False)
y = Variable(torch.randn(N, D_out).type(dtype), requires_grad=False)

# Create random Tensors for weights, and wrap them in Variables.
w1 = Variable(torch.randn(D_in, H).type(dtype), requires_grad=True)
w2 = Variable(torch.randn(H, D_out).type(dtype), requires_grad=True)

learning_rate = 1e-6
for t in range(500):
    # Construct an instance of our MyReLU class to use in our network
    relu = MyReLU()

    # Forward pass: compute predicted y using operations on Variables; we compute
    # ReLU using our custom autograd operation.
    y_pred = relu(x.mm(w1)).mm(w2)

    # Compute and print loss
    loss = (y_pred - y).pow(2).sum()
    print(t, loss.data[0])

    # Use autograd to compute the backward pass.
    loss.backward()

    # Update weights using gradient descent
    w1.data -= learning_rate * w1.grad.data
    w2.data -= learning_rate * w2.grad.data

    # Manually zero the gradients after updating weights
    w1.grad.data.zero_()
    w2.grad.data.zero_()
```

## torch.nn
### Container
#### Module

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

## torch.optim

## torchvision
### torchvision.datasets
torchvision提供了很多数据集
``` python
import torchvision
print(torchvision.datasets.__all__)
```
> ('LSUN', 'LSUNClass', 'ImageFolder', 'DatasetFolder', 'FakeData', 'CocoCaptions',     'CocoDetection', 'CIFAR10', 'CIFAR100', 'EMNIST', 'FashionMNIST', 'MNIST', 'STL10',     'SVHN', 'PhotoTour', 'SEMEION', 'Omniglot')

### CIFAR10
#### 原型
``` python
calss torchvision.datasets.CIFAR10(root, train=True, transform=None, target_transform=None, download=False)
```

#### 参数
root (string) – cifar-10-batches-py的存放目录或者download设置为True时将会存放的目录。
train (bool, optional) – 设置为True的时候, 从training set创建dataset, 否则从test set创建dataset.
transform (callable, optional) – 输入是一个 PIL image，返回一个transformed的版本。如，transforms.RandomCrop
target_transform (callable, optional) – A function/transform that takes in the target and transforms it.
download (bool, optional) – If true, downloads the dataset from the internet and puts it in root directory. If dataset is already downloaded, it is not downloaded again.
#### 例子
``` python
import torchvision
trainset = torchvision.datasets.CIFAR100(root="./datasets", train=True, transform=    None, download=True)
testset = torchvision.datasets.CIFAR100(root="./datasets", train=False, transform=    None, download=True)

```

### torchvision.models
模型
### torchvision.transforms
transform
### torchvision.utils
一些工具包

## torch.utils.data
### Dataloader
#### 原型
``` python
class torch.utils.data.DataLoader(dataset, batch_size=1, shuffle=False, sampler=None, batch_sampler=None, num_workers=0, collate_fn=<function default_collate>, pin_memory=False, drop_last=False, timeout=0, worker_init_fn=None)
```
#### 参数
dataset (Dataset) – 从哪加载数据
batch_size (int, optional) – batch大小 (default: 1).
shuffle (bool, optional) – 每个epoch的数据是否打乱 (default: False).
sampler (Sampler, optional) – 定义采样策略。如果指定这个参数, shuffle必须是False.
batch_sampler (Sampler, optional) – like sampler, but returns a batch of indices at a time. Mutually exclusive with batch_size, shuffle, sampler, and drop_last.
num_workers (int, optional) – 多少个子进程用来进行数据加载。0代表使用主进程加载数据 (default: 0)
collate_fn (callable, optional) – merges a list of samples to form a mini-batch.
pin_memory (bool, optional) – If True, the data loader will copy tensors into CUDA pinned memory before returning them.
drop_last (bool, optional) – set to True to drop the last incomplete batch, if the dataset size is not divisible by the batch size. If False and the size of dataset is not divisible by the batch size, then the last batch will be smaller. (default: False)
timeout (numeric, optional) – if positive, the timeout value for collecting a batch from workers. Should always be non-negative. (default: 0)
worker_init_fn (callable, optional) – If not None, this will be called on each worker subprocess with the worker id (an int in [0, num_workers - 1]) as input, after seeding and before data loading. (default: None)

#### 例子
``` python
import torch
import torchvision
dataset = torchvision.datasets.CIFAR100(root='./data', train=True, download=True, transform=None)
train_loader = torch.utils.data.DataLoader(dataset, bath_size=16, shuffle=False, num_worker=2)
```
#### 如何访问DataLoader返回值
train_loader不是整数，所以不能用range，这里用enumerate()，i是
``` python
for i, data in enumerate(train_loader):
    images, labels = data
```


## torch.distributed

## 参考文献
1.https://pytorch.org/docs/stable/torch.html
2.https://pytorch.org/docs/stable/nn.html
3.http://pytorch.org/tutorials/beginner/pytorch_with_examples.html
4.https://discuss.pytorch.org/t/distributed-model-parallelism/10377
5.https://ptorch.com/news/40.html
6.https://discuss.pytorch.org/t/distributed-data-parallel-freezes-without-error-message/8009
7.https://discuss.pytorch.org/t/runtimeerror-cudnn-status-arch-mismatch/3580
8.https://discuss.pytorch.org/t/error-when-using-cudnn/577/7
9.https://www.zhihu.com/question/67209417
