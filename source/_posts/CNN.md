---
title: CNN
date: 2019-03-13 15:21:27
tags:
 - CNN
 - 卷积神经网络
 - alexnet
categories: 机器学习
---

## Alexnet(2012)
### 概述
作者训练了一个深度卷积神经网络用来将Imagenet数据集中1000个类别中的120万张图片进行分类。整个网络结构包含五个卷积层，三个全连接层，以及一个1000-way的softmax层，整个网络共有6000万参数，65000个神经元。此外，作者提出了一些方法用来提高性能和减少训练的时间，并且介绍了一些防止过拟合的技巧。最后，在测试集上，它们跑出$37.5%$的top-1 error以及$17.0%$的top-5 error。

### 存在的问题
1.之前的数据集太小，都是数以万计的，需要更大的数据集。

### 创新
#### ReLU非线性激活函数
##### 作用
作者说实验表明ReLU可以加速训练过程。

##### saturating nonlinearity
一个饱和的激活函数会将输出挤压到一个区间内。
> A saturating activation function squeezes the input.

**定义**
f是non-saturating 当且仅当$|lim_{z\rightarrow -\infty} f(z)| \rightarrow + \infty$或者$|lim_{z\rightarrow +\infty} f(z)| \rightarrow + \infty$
f是saturating 当且仅当f不是non-saturating
**例子**
ReLU就是non-saturating nonlinearity的激活函数，因为$f(x) = max(0, x)$，如下图所示。
![relu](/relu.png)
当$x$趋于无穷时，$f(x)$也趋于无穷。
sigmod和tanh是saturating nonlinearity激活函数，如下图所示。
![sigmo](/sigmod.png)
![tanh](/tanh.png)


#### 多块GPU并行
作者使用了两块GPU一块运行，每个GPU中的参数个数是一样的，在一些特定层中，两个GPU中的参数信息可以进行通信。

#### Overlapping Pooling

就是Pooling kernel的size要比stride大。比如一个$12\times 12$的图片，用$5\times 5$的pooling kernel，步长为$3$，步长要比kernel核小，即$3$比$5$小。
为什么这能减小过拟合？
- 可能是减小了Pooling过程中信息的丢失。

> If the pooling regions do not overlap, the pooling regions are disjointed and if that is the case, more information is lost in each pooling layer. If some overlap is allowed the pooling regions overlap with some degree and less spatial information is lost in each layer.[4]

#### 数据增强
防止过拟合
##### 裁剪和翻转
输入是$256\times 256 \times 3$的图像。
训练：对每张图片都提取多个$224\times 224$大小的patch，这样子总共就多产生了$(256-224)\times (256-224) = 1024$个样本，然后对每个patch做一个水平翻转，就有$1024\times 2 = 2048$个样本。
测试：通过对每张图片裁剪五个（四个角落加中间）$224\times 224$的patches，并且对它们做翻转，也就是有$10$个patches，网络对十个patch的softmax层输出做平均作为预测结果。
##### 在图片上调整RGB通道的密度
使用PCA对RGB值做主成分分析。对于每张训练图片，加上主成分，其大小正比于特征值乘上一个均值为$0$，方差为$0.1$的高斯分布产生的随机变量。对于一张图片$x,y$点处的像素值$I_{xy}=[I_{xy}^R, I_{xy}^G,I_{xy}^B]^T$，加上$\[\bold{p_1},\bold{p_2},\bold{p_3}\]\[\alpha_1\lambda_1,\alpha_2\lambda_2,\alpha_3\lambda_3\]$，其中$\[\bold{p_1},\bold{p_2},\bold{p_3}\]$是特征向量，$\lambda_i$是特征值，$\alpha_i$就是前面说的随机变量。

#### Dropout
通过学习鲁棒的特征防止过拟合。
在训练的时候，每个隐藏单元的输出有$p$的概率被设置为$0$，在该次训练中，如果这个神经元的输出被设置为$0$，它就对loss函数没有贡献，反向传播也不会被更新。对于一层有$N$个神经单元的全连接层，总共有$2^N$种神经元的组合结果，这就相当于训练了一系列共享参数的模型。
在测试的时候，所有隐藏单元的输出都不丢弃，但是会乘上$p$的概率，相当于对一系列集成模型取平均。具体可见[dropout](https://mxxhcm.github.io/2019/03/23/神经网络-dropout/)
在该模型中，作者在三层全连接层的前两层输出上加了dropout。


#### 局部响应归一化(Local Response Normalizaiton)
事实上，后来发现这个东西没啥用。但是这里还是给出一个公式。

$$ b^i_{x,y} = \frac{a^i_{x,y}}{(k+\alpha \sum^{min(N-1,\frac{i+n}{2})}_{j=max(0,\frac{i-n}{2})}(a^j_{x,y})^2)^{\beta}}$$
其中$a^i_{x,y}$是在点$(x,y)$处使用kernel $i$之后，在经过ReLU激活函数。$k,n,\alpha,\beta$是超参数。
> It seems that these kinds of layers have a minimal impact and are not used any more. Basically, their role have been outplayed by other regularization techniques (such as dropout and batch normalization), better initializations and training methods.

### 整体架构
#### 目标函数
多峰logistic回归。
#### 并行框架
下图是并行的架构，分为两层，上面一层用一个GPU，下面一层用一个GPU，它们只在第三个卷积层有交互。
![alexnet](alexnet.png)
#### 简化框架
下图是简化版的结构，不需要使用两个GPU。
![alexnet_simple](alexnet_simple.png)

#### 数据流（简化框架）
输入是$224\times 224 \times 3$的图片，第一层是$96$个stride为$4$的$11\times 11\times 3$卷积核构成的卷积层，输出经过max pooling(步长为2，kernel size为3)输入到第二层；第二层有$256$个$5\times 5\times 96$个卷积核，输出经过max pooling(步长为2，kernel size为3)输入到第三层；第三层到第四层，第四层到第五层之间没有经过pooling和normalization)，第三层有384个$3\times 3\times 256$个卷积核，第四层有$384$个$3\times 3\times 384$个卷积核，第五层有$256$个$3\times 3\times 384$个卷积核。然后接了两个$2048$个神经元的全连接层和一个$1000$个神经元的全连接层。

### 实验
#### Datasets
ILSVRC-2010

#### Baselines
- Sparse coding
- SIFT+FV
- CNN

#### Metric
- top-1 error rate
- top-5 error rate

## 代码
pytorch实现
https://github.com/mxxhcm/myown_code/blob/master/CNN/alexnet.py

## Vggnet(2013)
### 概述
### 存在的问题
### 方案
#### 背景
#### 算法 
#### 代码

## 
### 概述
### 存在的问题
### 方案
#### 背景
#### 算法 
#### 代码

##
### 概述
### 存在的问题
### 方案
#### 背景
#### 算法 
#### 代码

##
### 概述 ### 存在的问题
### 方案
#### 背景
#### 算法 
#### 代码

##
### 概述
### 存在的问题
### 方案
#### 背景
#### 算法 
#### 代码


## 参考文献
1.https://stats.stackexchange.com/a/174438
2.https://www.zhihu.com/question/264163033/answer/277481519
3.https://stats.stackexchange.com/questions/145768/importance-of-local-response-normalization-in-cnn
4.https://stats.stackexchange.com/a/386304
5.https://blog.csdn.net/luoyang224/article/details/78088582/
