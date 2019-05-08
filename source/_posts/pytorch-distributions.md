---
title: pytorch distributions
date: 2019-05-08 21:53:46
tags:
 - pytorch
 - python
categories: pytorch
---
## torch.distributions
这个库和gym.space库很相似，都是提供一些分布，然后从中采样。
常见的有ExponentialFamily,Bernoulli,Binomial,Categorical,Exponential,Gamma,Independent,Laplace,Multinomial,MultivariateNormal。这里不做过程陈述，可以看[gym](http://localhost:4000/2019/04/12/gym%E4%BB%8B%E7%BB%8D/)中。
### Categorical
对应tensorflow中的[tf.multinomial](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_multinominal.py)。
类原型：
``` python
CLASS torch.distributions.categorical.Categorical(probs=None, logits=None, validate_args=None)
```
参数probs只能是$1$维或者$2$维，而且必须是非负，有限非零和的，然后将其归一化到和为$1$。
这个类和torch.multinormal是一样的，从$\{0,\cdots, K-1\}$中按照probs的概率进行采样，$K$是probs.size(-1)，即是size()矩阵的最后一列，$2$维时把第$1$维当成了batch。

举一个简单的例子，[代码]()。
``` python
import torch.distributions as diss
import torch

torch.manual_seed(5)

m = diss.Categorical(torch.tensor([0.25, 0.25, 0.25, 0.25 ]))
for _ in range(5):
    print(m.sample())

m = diss.Categorical(torch.tensor([[0.5, 0.25, 0.25], [0.25, 0.25, 0.5]]))
for _ in range(5):
    print(m.sample())

```

输出结果如下：
> tensor(2)
tensor(1)
tensor(1)
tensor(1)
tensor(1)
tensor([2, 2])
tensor([1, 2])
tensor([0, 1])
tensor([0, 2])
tensor([0, 0])

作为对比，gym.spaces.Discrete示例如下：
``` python
from gym import spaces

# 1.Discrete
# 取值是{0, 1, ..., n - 1}
dis = spaces.Discrete(5)
dis.seed(5)
for _ in range(5):
    print(dis.sample())
```
输出结果是：
> 3
0
1
0
4


