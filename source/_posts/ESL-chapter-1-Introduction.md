---
title: ESL chapter 1 Introduction
date: 2019-01-05 09:46:39
tags:
 - 机器学习
categories: 机器学习
mathjax: true
---

## 引言
这本书主要介绍的是统计学习。一些典型的学习问题如下：
- 基于一个心脏病患者的饮食，临床检测等等，去预测一个这个因为心脏病住院的人会不会第二次患心脏病。
- 基于一个公司的运行状况或一些经济数据，去预测未来六个月股票的价格。
- 从一个手写字母图像中识别出来其中的字母。
- 从一个糖尿病(diabetic)患者血液的红外吸收频谱去预测他的血糖(glucose)含量。
- 基于人口统计(demographic)和临床检测，分析前列腺癌的致病因素。 

在一个典型的学习场景下，我们通常有一些定量的结果(outcome measurement)，如上面例子中的股票价格或者分类问题中问题的类别，我们希望基于一系列的特征进行预测。
接下来给了几个真实的学习问题的示例。下面就简要介绍一下这几个例子。

### 邮件分类
给定一封邮件，邮件分类的目标就是根据邮件的特征去判断这封邮件是正常邮件还是垃圾邮件。这是监督学习中的二分类问题，因为该问题有ouputs，且只有两个类别。

### 前列腺癌(prostate cancer)
该问题的目标是给定一系列临床检测，如记录癌症量(log cancer volume)，去预测前列腺特异性抗原(prstate specific antigen)的数量。该问题是监督学习中的回归问题，因为结果(outcome measurement)是定量的(quatitative)。

### 手写数字识别
给定一个手写数字的图片，该问题的目标是识别出图片中的数字。

### DNA Expression Microarrays
这个问题是通过基因数组去学习基因和不同基因样本之间的关系，一些典型的问题是：
1. 哪些样本之间是相似的？在不同的基因之间都相似。
2. 哪些基因是相似的？在不同的样本之间都相似。
3. 一些特定的基因对于特定的癌症患者表达是达不是很明显？

这个问题可以看成回归问题，或者更有可能是无监督问题。

## 本书结构
第一章就是本章。第二章讲监督学习的介绍。第三章和第四章介绍线性回归和分类。第五章介绍仿样(splines)，小波(wavelets)，正则化(regularization)和惩罚(penalization)。第六章介绍核方法(kernel methods)和局部回归(local regression)。第七章将模型估计和选择(model assessment and selection)，涉及到偏置(bias)和方差(variance)，过拟合(overfitting)以及交叉验证(cross-validation)等等。第八章讲模型推理。第十章讲boosting。
第九到十三章讲监督学习的一系列结构化方法。十四章介绍非监督学习。十五和十六章分别介绍随机森林(random forests)和集成学习(ensemble learning)。第十七章介绍无向图(undirected graphical models)。第十八章介绍高维问题。
第一到四章是基础最好按顺序阅读，第七章也是。其他的可以不按顺序。
