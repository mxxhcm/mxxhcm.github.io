---
title: CNN
date: 2019-03-13 15:21:27
tags:
 - CNN
 - 卷积神经网络
 - alexnet
categories: 机器学习
mathjax: true
---

## Alexnet(2012)
### 概述
作者提出了一个卷积神经网络架构对Imagenet中$1000$类中的$120$万张图片进行分类。网络架构包含$5$个卷积层，$3$个全连接层，和一个$1000$-way的softmax层，整个网络共有$6000$万参数，$65000$个神经元。作者提出了一些方法提高性能和减少训练的时间，并且介绍了一些防止过拟合的技巧。最后在imagenet测试集上，跑出$37.5%$的top-1 error以及$17.0%$的top-5 error。

### 问题
1.数据集太小，都是数以万计的，需要更大的数据集。

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
目的：防止过拟合
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

$$ b^i_{x,y} = \frac{a^i_{x,y}}{(k+\alpha \sum^{min(N-1,\frac{i+n}{2})}\_{j=max(0,\frac{i-n}{2})}(a^j\_{x,y})^2)\^{\beta}}$$
其中$a^i\_{x,y}$是在点$(x,y)$处使用kernel $i$之后，在经过ReLU激活函数。$k,n,\alpha,\beta$是超参数。
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

### 代码
pytorch实现
https://github.com/mxxhcm/myown_code/blob/master/CNN/alexnet.py

## OverFeat(2013)

### 概述
本文提出了一个可用于classification, localization和detection等任务的CNN框架。
ImageNet数据集中大部分选择的是几乎填满了整个image中心的object，image中我们感兴趣的objects的大小和位置也可能变化很大。为了解决这个问题，作者提出了三个方法：
1. 用sliding window和multiple scales在image的多个位置apply ConvNet。即使这样，许多window中可能包含能够完美识别object类型的一部分，比如一个狗头。。。最后的结果是classfication很好，但是localization和detection结果很差。
2. 训练一个网络不仅仅预测每一个window的category distribution，还预测包含object的bounding box相对于window的位置和大小。
3. 在每个位置和大小累加每个category的evidence


### Vision任务
classification，localization和detection。classification和localization通常只有一个很大的object，而detection需要找到很多很小的objects。
classification任务中，每个image都有一个label对应image中主要的object的类型。为了找到正确的label，每个图片可以猜$5$次（图片中可能包含了没有label的数据）。localization任务中，不仅要给出label，还需要找到这个label对应的bouding box，bounding box和groundtruth至少要有$50$匹配，label和bounding box也需要匹配。detection和localization不同的是，detection任务中可以有任何数量的objects，false positive会使用mean average precison measure。localization任务可以看成classification到detection任务的一个中间步。

### Classification
#### training
- datset
Image 2012 trainign set（1.2million iamges，C=$1000$ classes)。
- data argumented
对每张图片进行下采样，所以每个图片最小的dimension需要是$256$。
提取$5$个random crops以及horizaontal flips，总共$10$个$221\times 221$的图片
- batchsize
$128$
- 初始权重
$(\mu, \sigma)= (0, 1\times 10\^{-2})$
- momentum
0.6
- l2 weigth decay
$1\times 10\^{-5}$
- lr
初始是$5\times 10\^{-2}$，在$(30,50,60,70,80)$个epoches后，乘以$0.5$
- non-spatial
这个说的是什么呢，在test的时候，会输出多个output maps，对他们的结果做平均，而在training的时候，output maps是$1\times 1$。

#### fast model架构
下图展示的是fast model，spatial input size在train和test时候是不同的，这里展示的是train时的spatial seize。layer 5是最上层的CNN，receptive filed最大。后续是FC layers，在test时候使用了sliding window。在spatial设置中，FC-layers可以查看$1\times 1$的卷积。
![overfeat_fast](overfeat_fast.png)
![overfeat_accuracy](overfeat_accuracy.png)

#### 多scale classification 
alexnet中，对一张照片的$10$个views（中间，四个角和horizontal flip)的结果做了平均，这种方式可能会忽略很多趋于，同时如果不同的views有重叠的话，计算很redundant。此外，alexnet中只使用了一个scale。
作者对每个iamge的每一个location和多个scale都进行计算。


## Vggnet(2013)
### 概述
这篇文章主要研究了CNN深度对大规模图像识别问题精度的影响。通过使用$3\times 3$的filters，增加CNN的深度，提高识别精度。

### 方案
#### 架构
**训练**，输入$224\times 224$大小的RGB图片。对每张图片减去训练集上所有图片RGB 像素的均值。预处理后的图片被输入多层CNN中，CNN的filter是$3\times 3$的，作何也试了$1\times 1$的filter，相当于对输入做了一个线性变换，紧跟着一个non-linear 激活函数。stride设为$1$，添加padding使得卷积后的输出大小不变。同时使用了$5$个max-pooling层（并不是每一层cnn后面都有max-pooling)，max-pooling在$2\times 2$大小的窗口上，stride是$2$。
多层CNN后面接的是三个FC layers，前两个是$4096$单元，最后一层是$1000$个单元的softmax。所有隐藏层都使用ReLu非线性激活函数。

#### 配置
这篇文章给出了五个网络架构，用$A-E$表示，它们只有在深度上有所不同：从$11$层($8$个conv layers和$3$个FC layers)到$19$层（$16$个conv layers和$3$个FC layers）。Conv layers的channels很小，从第一层的$64$，每过一个max pooling layers，变成原理啊的两倍，直到$512$。具体如下表所示。
![vgg_conf](vgg_conf.png)
网络的参数个数如下表所示。
![vgg_weights_num](vgg_weights_num.png)
网络$A$的参数计算：
\begin{align\*}
64\times 3\times 3\times 3 + \\\\
128\times 3\times 3\times 64 + \\\\ 
256\times 3\times 3\times 128 + \\\\
256\times 3\times 3\times 256 + \\\\
512\times 3\times 3\times 256 + \\\\
512\times 3\times 3\times 512 + \\\\
2\times 512\times 3\times 3\times 512 + \\\\
7\times 7\times 512\times 4096 + \\\\
4096\times 4096 + \\\\
4096\times 1000 = \\\\
132851392
\end{align\*}
网络$B$的参数计算：
\begin{align\*}
64\times 3\times 3\times 3 + \\\\
128\times 3\times 3\times 64 + \\\\ 
128\times 3\times 3\times 128 + \\\\ 
256\times 3\times 3\times 128 + \\\\
256\times 3\times 3\times 256 + \\\\
256\times 3\times 3\times 256 + \\\\
512\times 3\times 3\times 256 + \\\\
512\times 3\times 3\times 512 + \\\\
2\times 512\times 3\times 3\times 512 + \\\\
7\times 7\times 512\times 4096 + \\\\
4096\times 4096 + \\\\
4096\times 1000 = \\\\
133588672
\end{align\*}
其实主要的网络参数还是在全连接层，$7\times 7\times 512\times 4096=102760448
$。

#### 讨论
1. 为什么要用三个$3\times 3$的conv layers替换$7\times 7$个conv layers？
- 使用三个激活函数而不是一个，让整个决策更discriminative。
- 减少了网络参数，三个有$C$个通道的$3\times 3$conv layers,总的参数是$3\tims(3^2C^2)=27C^2$，而一个$C$通道的$7\times 7$ conv layers，总参数是$49C^2$。可以看成是一种正则化。
2. $1\times 1$ conv layers用来增加非线性程度，本文中使用的$1\times 1$的conv layers可以看成加了非线性激活函数的投影。

### 分类框架
#### training
- 目标函数
多峰logistic regression
- 训练方法
mini-batch gradient descent with momentum
- batch size
256
- momentum 
0.9
- 正则化
$L_2$参数正则化(5\codt 10\^{-4})
0.5 dorpout 用于前两个FC layers
- lr
初始值为$10\^{-2}$，当验证集的accuracy不再提升时，除以$10$。学习率总共降了$3$次，$370K$次迭代后停止。
- 图像预处理
从rescaled中随机cropped $224\times 224$的RGB图像。
使用alexnet中的随机horizontal flipping和随机RGB colour shift。
- iamge rescale
用$S$表示training image的小边的大小，$S$也叫作train sacle。网络的输入是从training image中cropped得到的$224\times 224$的图像。所以只要$S$取任何不小于$224$的值即可，如果$S=224$，那么crop在统计上会captuer整个图片，完全包含training image最小的那边；$S\>\>224$的时候，crop会产生很小一部分的图像。
作者尝试了固定$S$和不固定的$S$。对于固定$S$，设置$S=256$和$S=384$，首先在$S=256$上训练，然后用$S=256$训练的参数初始化$S=384$的参数，使用更小的初始学习率$10\^{-3}$。不固定$S$时，$S$从$\[S\_{min}, S\_{max}\](S\_{max}=512,S\_{min}=256)$任意采样，然后crop。
- VGG vs alexnet
VGG参数多，深度深，但是收敛快，原因：
1. 更小的filter带来的implicit regularisation
2. 某些层的预先初始化。
这个解决的是网络深度过深，某些初值使得网络不稳定的问题。解决方法：先随机初始化不是很深的网络A，进行训练。在训练更深网络的时候，使用A网络的值初始化前$4$个卷基层和最后三个FC layers。随机初始化的网络参数，从均值为$0$，方差为$10\^{-2}$的高斯分布中采样得到。

#### testing
测试的时候先把input image的窄边缩放到$Q$，$Q$也叫test scale，$Q$和$S$不一定需要相等。

### classfication
ILSVRC-2012，training($1.3M$张图片)，validation($50K张$)，testing($100K$张)
两个metrics：top-1和top-5 error。top-1 error是multi-class classification error，不正确分类图像占的比例；top-5 error是预测的top-5都不是ground-truth。

#### single scale evaluation 
$S$固定时，设置test image size $Q=S=256$；
$S$抖动时，设置test image size $Q=0.5(S\_{min}+S\_{max})=0.5(256+512)=384$，$S\in \[S\_{min},S\_{max}\]$。

#### multi scale evaluation
用同一个模型对不同rescaled大小的图片多次test，即对于不同的$Q$。
固定$S$时，在三个不同大小的test image size $Q=\{S-32,S,S+32\}$评估。
$S$抖动时，模型是在$S\in \[S\_{min},S\_{max}\]$上训练的，在$Q=\{S\_{min}, 0.5(S\_{min}+S\_{max}), S\_{max}$上进行test。

#### 多个crop evaluation
这个是为了和alexnet做对比，alexnet网络在testing时，对每一张图片都进行多次cropped，对testing的结果做平均。

#### convnet funsion
之前作者的evaluation都是在单个的网络上进行的，作者还试了将不同网络的softmax输出做了平均。

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
