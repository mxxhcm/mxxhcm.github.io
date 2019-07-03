---
title: 'python-anaconda报错ImportError: No module named conda.cli'
date: 2019-07-03 19:54:24
tags:
 - python
 - anaconda
categories: python
---

## 问题描述
anaconda的python版本是3.7，执行了conda install python=3.6之后，运行conda命令出错。报错如下：
``` txt
from conda.cli import main 
ModuleNotFoundError: No module named 'conda'
```

## 解决方案
找到anaconda安装包，加一个-u参数，如下所示。重新安装anaconda自带的package，自己安装的包不会丢失。
~$:sh xxx.sh -u

## 参考文献
1.https://blog.csdn.net/u014432608/article/details/79066813
