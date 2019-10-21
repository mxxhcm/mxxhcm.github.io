---
title: maximum likelyhood estimation
date: 2019-01-20 15:22:45
tags:
 - 参数估计
 - 点估计
 - 最大似然估计
 - 概率论
 - 机器学习
categories: 概率论
mathjax: true
---

## 最大似然估计
前提：数据的分布已知（如服从高斯分布或者指数分布），但是分布参数未知。例如某学校学生的身高服从高斯分布$N(\mu, \sigma^2 )$，$\mu$和$\sigma^2 $未知。现随机抽取$200$个学生的身高，估计该学校学生身高的均值$\mu$和方差$\sigma^2 $。
概率密度：$p(x|\theta)$
样本集：$X=(x_1, x_2,\cdots, x_N), N=200$，$x_i$为第$i$个人的身高。
假设分布：$N(\mu, \sigma^2 )$，$\mu, \sigma^2 $未知

### 联合概率密度函数
样本集中的$N$个样本是独立同分布的，它们的联合概率可以表示为：
$$L(\theta; X) = L(x_1, \cdots, x_n;\theta) = \prod_{i=1}^{N} p(x_i|\theta), \theta \in \Theta$$

### 取对数
因为$L$中包含乘法，不方便求导，可以对其取log，不改变函数的单调性，并且方便计算：
$$ln\ L(\theta; X) = ln\ L(\theta; x_1, \cdots, x_n) = \sum_{i=1}^N ln\ p(x_i|\theta), \theta \in \Theta$$

### 求极值
计算偏导数，令其等于$0$，取函数的极值点，因为只有一个极值点，所以一定是最大值点，即
$$\hat{\theta} = arg\ max_{\theta} ln\ L(\theta; x)$$
求偏导等于$0$即：
$$\frac{\partial ln\ L(\theta; X)}{\partial \theta} = \sum_{i=1}^N \frac{\partial ln\ p(x_i;\theta)}{\partial \theta}, \theta={\mu, \sigma^2}$$

## 最大似然估计求解高斯分布
若$p$为高斯分布，即$p(x; \theta) = \frac{1}{\sqrt{2\pi} \sigma} e\^{-\frac{(x-\mu)^2 }{2 \sigma^2 } } $，$ln\ p(x;\theta) = -ln\ \sqrt{2\pi } - ln\ \sigma - \frac{(x_i-\mu)^2 }{ 2\sigma^2 }$
则：
$$ln\ L(\theta; X) = ln\ L(\theta; x_1, \cdots, x_n) = \sum_{i=1}^N ln\ p(x_i;\theta) = \sum_{i=1}^N \left(-ln\ \sqrt{2\pi} - ln\ \sigma - \frac{(x_i-\mu)^2 }{ 2\sigma^2 }\right), \theta \in \Theta$$

### 求解$\mu$
对$ \mu $ 求偏导得：
$$\frac{\partial ln\ L(\mu; X)}{\partial \mu} = \sum_{i=1}^N \frac{\partial ln\ p(x_i;\mu)}{\partial \mu} = \sum_{i=1}^N -\frac{2(x_i-\mu) }{2\sigma^2 } $$
令其等于$0$，即
$$\sum_{i=1}^N(x_i-\mu) = 0$$
解得$$ \mu = \frac{\sum_{i=1}^N x_i }{N}$$
### 求解$\sigma^2 $
对$ \sigma^2 $求偏导，令$t=\sigma^2 $，则$ln\ p(x;\sigma^2 ) = ln\ p(x;t) = -ln\ \sqrt{2\pi} - ln\ \sqrt{t} - \frac{(x_i-\mu)^2 }{2t}$，有：
$$ \frac{\partial ln\ L(t; X)}{\partial t} = \sum_{i=1}^N \frac{\partial ln\ p(x_i;t)}{\partial t} = \sum_{i=1}^N\left( -\frac{1}{2t} + \frac{(x-\mu)^2 }{2t^2 }\right) $$
令其等于$0$，即
$$ \sum_{i=1}^N\left( -\frac{1}{2t} + \frac{(x-\mu)^2 }{2t^2 }\right) = 0 $$
解得：
$$ \sigma^2 = t = \frac{\sum_{i=1}^N (x_i) -\mu)^2 }{N} $$



## 参考文献
