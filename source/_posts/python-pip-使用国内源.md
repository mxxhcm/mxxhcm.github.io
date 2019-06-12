---
title: pip 使用国内源
date: 2019-03-07 09:56:54
tags:
 - python
 - linux
categories: linux
---

## 暂时使用国内pip源
使用清华源
~\$:pip install -i https://pypi.tuna.tsinghua.edu.cn/simple package-name
使用阿里源
~\$:pip install -i https://mirrors.aliyun.com/pypi/simple package-name

## 将国内pip源设为默认
~\$:pip install pip -U
~\$:pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
~\$:pip config set global.timeout 60
> Writing to /home/username/.config/pip/pip.conf

### 查看pip配置文件
~\$:find / -name pip.conf
我的是在/home/username/.config/pip/pip.conf


## 参考文献
1.https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
