---
title: pytorch 常见问题
date: 2019-05-08 21:52:18
tags:
 - pytorch
 - 常见问题
 - python
categories: pytorch
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
10.https://pytorch.org/docs/stable/distributions.html#categorical
