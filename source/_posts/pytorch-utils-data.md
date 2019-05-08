---
title: pytorch utils data
date: 2019-05-08 21:56:15
tags:
 - pytorch
 - python
categories: pytorch
---

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


