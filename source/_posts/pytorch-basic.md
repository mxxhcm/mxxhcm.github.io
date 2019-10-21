---
title: pytorch
date: 2019-05-08 22:07:42
tags:
 - pytorch
 - python
categories: pytorch
---

## torch
torch提供了很多基础操作，包括数学操作等等。

## torch.cat
### 函数原型
将多个tensor在某一个维度上（默认是第0维）拼接到一起（除了拼接的维度上，其他维度的shape必须一定），最后返回一个tensor。
torch.cat(tensors, dim=0, out=None) → Tensor
> Concatenates the given sequence of seq tensors in the given dimension. All tensors must either have the same shape (except in the concatenating dimension) or be empty.

### 参数
tensors (sequence of Tensors) – 任意类型相同python序列或者tensor
dim (int, optional) - 在第几个维度上进行拼接(只有在拼接的维度上可以不同，其余维度必须相同。
out (Tensor, optional) – 输出的tensor

### 例子
```python
import torch

x1 = torch.randn(3, 4, 4)
x2 = torch.randn(3, 1, 4)
x = torch.cat([x1, x2], 1)
print(x.size())
```
输出如下：
> torch.Size([3, 5, 4])

## torch中图像(img)格式
torch中图像的shape是('RGB',width, height)，而numpy和matplotlib中都是(width, height, 'RGB')
matplotlib.pyplot.imshow()需要的参数是图像矩阵，如果矩阵中是整数，那么它的值需要在区间[0,255]之内，如果是浮点数，需要在[0,1]之间。
> Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers).

## 参考文献
1.https://pytorch.org/docs/stable/torch.html

