---
title: pytorch optim
date: 2019-05-08 21:58:19
tags:
 - pytorch
 - python
categories: pytorch
---

## torch.optim
### 基类class Optimizer(object)
Optimizer是所有optimizer的基类。
调用任何优化器都要先初始化Optimizer类，这里拿Adam优化器举例子。Adam optimizer的init函数如下所示：
``` python
class Adam(Optimizer):
	"""
        params (iterable): iterable of parameters to optimize or dicts defining
            parameter groups
        lr (float, optional): learning rate (default: 1e-3)
        betas (Tuple[float, float], optional): coefficients used for computing
            running averages of gradient and its square (default: (0.9, 0.999))
        eps (float, optional): term added to the denominator to improve
            numerical stability (default: 1e-8)
        weight_decay (float, optional): weight decay (L2 penalty)
        amsgrad (boolean, optional): whether to use the AMSGrad variant of this

    """

    def __init__(self, params, lr=1e-3, betas=(0.9, 0.999), eps=1e-8,
                 weight_decay=0, amsgrad=False):
        if not 0.0 <= lr:
            raise ValueError("Invalid learning rate: {}".format(lr))
        if not 0.0 <= eps:
            raise ValueError("Invalid epsilon value: {}".format(eps))
        if not 0.0 <= betas[0] < 1.0:
            raise ValueError("Invalid beta parameter at index 0: {}".format(betas[0]))
        if not 0.0 <= betas[1] < 1.0:
            raise ValueError("Invalid beta parameter at index 1: {}".format(betas[1]))
        defaults = dict(lr=lr, betas=betas, eps=eps,
                        weight_decay=weight_decay, amsgrad=amsgrad)
        super(Adam, self).__init__(params, defaults)
```

上述代码将学习率lr,beta,epsilon,weight_decay,amsgrad等封装在一个dict中，然后将其传给Optimizer的init函数，其代码如下：

``` python
class Optimizer(object):
    .. warning::
        Parameters need to be specified as collections that have a deterministic
        ordering that is consistent between runs. Examples of objects that don't
        satisfy those properties are sets and iterators over values of dictionaries.

    Arguments:
        params (iterable): an iterable of :class:`torch.Tensor` s or
            :class:`dict` s. Specifies what Tensors should be optimized.
        defaults: (dict): a dict containing default values of optimization
            options (used when a parameter group doesn't specify them).
    """

    def __init__(self, params, defaults):
        self.defaults = defaults

        if isinstance(params, torch.Tensor):
            raise TypeError("params argument given to the optimizer should be "
                            "an iterable of Tensors or dicts, but got " +
                            torch.typename(params))

        self.state = defaultdict(dict)
        self.param_groups = []

        param_groups = list(params)
        if len(param_groups) == 0:
            raise ValueError("optimizer got an empty parameter list")
        if not isinstance(param_groups[0], dict):
            param_groups = [{'params': param_groups}]

        for param_group in param_groups:
            self.add_param_group(param_group)
```
从这里可以看出来，每个pytorch给出的optimizer至少有以下三个属性和四个函数：
属性：
- self.defaults # 字典类型，主要包含学习率等值
- self.state # defaultdict(\<class 'dict'\>, {}) state存放的是
- self.param_gropus # \<class 'list'\>:[]，prama_groups是一个字典类型的列表，用来存放parameters。

函数：
- self.zero_grad()  # 将optimizer中参数的梯度置零
- self.step()  # 将梯度应用在参数上
- self.state_dict() # 返回optimizer的state,包括state和param_groups。
- self.load_state_dict()  # 加载optimizer的state。
- self.add_param_group()  # 将一个param group添加到param_groups。可以用在fine-tune上，只添加我们需要训练的层数，然后其他层不动。

如果param已经是一个字典列表的话，就无需操作，否则就需要把param转化成一个字典param_groups。然后对param_groups中的每一个param_group调用add_param_group(param_group)函数将param_group字典和defaults字典拼接成一个新的param_group字典添加到self.param_groups中。


## 参考文献
1.https://pytorch.org/docs/stable/optim.html
