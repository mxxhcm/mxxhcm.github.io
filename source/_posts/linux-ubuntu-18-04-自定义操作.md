---
title: linux-ubuntu 18.04 自定义操作
date: 2019-05-22 21:22:29
tags:
 - linux
 - ubuntu 
categories: linux
mathjax: true
---

## dock 配置
### 基本设置
打开Settings >> Dock，可以设置dock的位置和大小，以及自动隐藏。这些是ubuntu安装的默认配置。

### dconf安装
为了更多的设置，需要安装dconf-editor
~\$:sudo apt install dconf-tools
按下Win键，搜索dconfig-editor，打开它。
找到org>>gnome>>shell>>extensions>>dash-to-dock，然后就可以修改相应的配置了。也可以在命令行中进行相应的设置，这里就不说了，可以查看参考文献尝试。



## 参考文献
1.https://linuxconfig.org/how-to-customize-dock-panel-on-ubuntu-18-04-bionic-beaver-linux
