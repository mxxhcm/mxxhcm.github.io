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
class torch.utils.data.DataLoader(
	dataset, 	# 从哪加载数据
	batch_size=1, 	# batch大小 (default: 1).
	shuffle=False,	# 每个epoch的数据是否打乱 (default: False).
	sampler=None, 	# 定义采样策略。如果指定这个参数, shuffle必须是False.
	batch_sampler=None, 
	num_workers=0,		# 多少个子进程用来进行数据加载。0代表使用主进程加载数据 (default: 0)
	collate_fn=<function default_collate>, 
	pin_memory=False, 
	drop_last=False, 
	timeout=0, 
	worker_init_fn=None
)
```

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


