---
title: pytorch Variable(torch.autograd.Variable)
date: 2019-05-08 21:54:53
tags:
 - pytorch
 - python
categories: pytorch
mathjax: true
---

### Variable(class torch.autograd.Variable)
#### 声明一个tensor
- torch.zeros
- torch.ones
- torch.rand
- torch.full()
- torch.empyt()
- torch.rand()
- torch.randn()
- torch.ones_like()
- torch.zeros_like()
- torch.randn_like()
- torch.Tensor


#### 代码示例
[代码地址](https://github.com/mxxhcm/code/blob/master/pytorch/pytorch_test/torch_tensor.py)
``` python
import torch

torch.manual_seed(5)
x = torch.empty(5, 3)
print(torch.empty(5, 3)) # construct a 5x3 matrix, uninitialized
# tensor([[4.6179e-38, 4.5845e-41, 4.6179e-38],
#         [4.5845e-41, 6.3010e-36, 6.3010e-36],
#         [2.5204e-35, 6.3010e-36, 1.0082e-34],
#         [6.3010e-36, 6.3010e-36, 6.6073e-30],
#         [6.3010e-36, 6.3010e-36, 6.3010e-36]])

print(torch.rand(3, 4))  # construct a 4x3 matrix, uniform [0,1] 
# tensor([[0.8303, 0.1261, 0.9075, 0.8199],
#         [0.9201, 0.1166, 0.1644, 0.7379],
#         [0.0333, 0.9942, 0.6064, 0.5646]])

print(torch.randn(5, 3)) # construct a 5x3 matrix, normal distribution
# tensor([[-1.4017, -0.7626,  0.6312],
#         [-0.8991, -0.5578,  0.6907],
#         [ 0.2225, -0.6662,  0.6846],
#         [ 0.5740, -0.5829,  0.7679],
#         [ 0.5740, -0.5829,  0.7679],

print(torch.randn(2, 3).type())
# torch.FloatTensor

print(torch.zeros(5, 3)) # construct a 5x3 matrix filled zeros
# tensor([[0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.]])

print(torch.ones(5, 3)) # construct a 5x3 matrix filled ones
# tensor([[1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.]])

print(torch.ones(5, 3, dtype=torch.long)) # construct a tensor with dtype=torch.long
# tensor([[1, 1, 1],
#         [1, 1, 1],
#         [1, 1, 1],
#         [1, 1, 1],
#         [1, 1, 1]])

print(torch.tensor([1,2,3])) # construct a tensor direct from data
# tensor([1, 2, 3])

print(x.new_ones(5,4)) # constuct a tensor has the same property as x
# tensor([[1., 1., 1., 1.],
#         [1., 1., 1., 1.],
#         [1., 1., 1., 1.],
#         [1., 1., 1., 1.],
#         [1., 1., 1., 1.]])


print(torch.full([4,3],9))  # construct a tensor with a value 
# tensor([[9., 9., 9.],
#         [9., 9., 9.],
#         [9., 9., 9.],
#         [9., 9., 9.]])

print(x.new_ones(5,4,dtype=torch.int)) # construct a tensor with the same property as x, and also can have the specified type.
# tensor([[1, 1, 1, 1],
#         [1, 1, 1, 1],
#         [1, 1, 1, 1],
#         [1, 1, 1, 1],
#         [1, 1, 1, 1]], dtype=torch.int32)

print(torch.randn_like(x,dtype=torch.float)) # construct a tensor with the same shape with x, 
# tensor([[ 0.4699, -1.9540, -0.5587],
#         [ 0.4295, -2.2643, -0.2017],
#         [ 1.0677,  0.3246, -0.0684],
#         [-0.9959,  1.1563, -0.3992],
#         [ 1.2153, -0.8115, -0.8848]])

print(torch.ones_like(x))
# tensor([[1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.],
#         [1., 1., 1.]])

print(torch.zeros_like(x))
# tensor([[0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.],
#         [0., 0., 0.]])


print(torch.Tensor(3,4))
# tensor([[-3.8809e-21,  3.0948e-41,  2.3822e-44,  0.0000e+00],
#         [        nan,  7.2251e+28,  1.3733e-14,  1.8888e+31],
#         [ 4.9656e+28,  4.5439e+30,  7.1426e+22,  1.8759e+28]])

print(torch.Tensor(3,4).uniform_(0,1))
# tensor([[0.8437, 0.1399, 0.2239, 0.3462],
#         [0.5668, 0.3059, 0.1890, 0.4087],
#         [0.2560, 0.5138, 0.1299, 0.3750]])

print(torch.Tensor(3,4).normal_(0,1))
# tensor([[-0.5490, -0.0838, -0.1387, -0.5289],
#         [-0.4919, -0.4646, -0.0588,  1.2624],
#         [ 1.1935,  1.5696, -0.8977, -0.1139]])

print(torch.Tensor(3,4).fill_(5))
# tensor([[5., 5., 5., 5.],
#         [5., 5., 5., 5.],
#         [5., 5., 5., 5.]])

print(torch.arange(1, 3, 0.4))
# tensor([1.0000, 1.4000, 1.8000, 2.2000, 2.6000])
```

## tensor的各种操作
``` python
import torch
a = torch.ones(2,3)
b = torch.ones(2,3)
```

### 加操作
``` python
print(a+b)                #方法1
c = torch.add(a,b)    #方法2
torch.add(a,b,result)    #方法3
a.add(b)                    #方法4,将a加上b，且a不变
a.add_(b)                #方法5,将a加上b并将其赋值给a
```
### 转置操作
```python
print(a.t())               # 打印出tensor a的转置
print(a.t_())                 #将tensor a 转置，并将其赋值给a
```

### 求最大行和列
``` python
torch.max(tensor,dim)
np.max(array,dim)
```

### 和relu功能比较类似。
``` python
torch.clamp(tensor, min, max,out=None)
np.maximun(x1, x2)  # x1 and x2 must hava the same shape
```

## tensor和numpy转化
### convert tensor to numpy
``` python
a = torch.ones(3,4)
b = a.numpy()
```
### convert numpy to tensor
``` python
a =  numpy.ones(4,3)
b = torch.from_numpy(a)
```

## Variable和Tensor
Variable
图1.Variable

### 属性
如图1,Variable wrap a Tensor,and it has six attributes,data,grad,requies_grad,volatile,is_leaf and grad_fn.We can acess the raw tensor through .data operation, we can accumualte gradients w.r.t this Variable into .grad,.Finally , creator attribute will tell us how the Variable were created,we can acess the creator attibute by .grad_fn,if the Variable was created by the user,then the grad_fn is None,else it will show us which Function created the Variable.
if the grad_fn is None,we call them graph leaves
``` python
Variable.shape  #查看Variable的size
Variable.size()
```

### parameters
``` python
torch.autograd.Variable(data,requires_grad=False,volatile=False)
```
requires_grad : indicate whether the backward() will ever need to be called

### backward
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

## 参考文献
1.https://pytorch.org/docs/stable/tensors.html
