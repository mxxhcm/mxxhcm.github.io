---
title: pytorch torchvision
date: 2019-05-08 21:55:57
tags:
 - pytorch
 - python
categories: pytorch
---
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


