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

## ubuntu 18.04合上笔记本盖子后不挂起
~\$:sudo apt install gnome-tweak-tool
~\$:gnome-tweaks
找到Power选项，设置Suspend when lapto lid is closed为OFF。[6]

## 显示cpu和gpu温度
### 安装lm-sensors
~\$:sudo apt-get install lm-sensors 
然后执行以下命令进行配置：
~\$:sudo sensors-detect
执行sensors命令获得各项硬件的温度
~\$:sensors

## 安装gnome-shell
### 安装gnome tweak tool
~\$:sudo apt install gnome-tweak-tool
gnome tweak 用来查看本地的gnome 插件。

### 从ubuntu 仓库安装extensions
ubuntu 提供了gnome-shell-extensions包，该包中有部分gnome扩展。然后可以使用gnome tweaks查看已经安装的程序。
~\$:sudo apt install gnome-shell-extensions

### 在浏览器上安装gnome shell integration插件
在firfox或者chrome上安装相应的gnome shell integration插件。
这个时候是不能添加插件的，因为还缺少一个东西，叫做native host connector
这种方法和从ubuntu仓库中装extension的不同之处是，ubuntu包中的扩展是固定的一部分，这中方法可以自定义安装。
安装完之后可以直接在浏览器的gnome shell integration插件上查看在浏览器上安装的gnome shell扩展，也可以使用gnome tweaks查看浏览器上安装的shell extensions。

### 安装chrome-gnome-shell native host connector
执行以下命令进行安装，chrome-gnome-shell并不是代表chrome浏览器的意思，用任何浏览器都要执行以下命令
~\$:sudo apt install chrome-gnome-shell

### 安装相应的插件
#### 命令行下
搜索
~\$:sudo apt search gnome-shell-extension
安装
~\$:sudo apt install gnome-shell-extension-package-name

#### 浏览器中
直接打开gnome shell extensions图形化界面进行搜索安装


### 插件推荐
- Coverflow Alt-Tab 按alt tab切换程序效果
[7]


## 修改主题
下载系统主题文件，解压缩，放置在/usr/share/themes文件夹下。然后在tweaks中的Apperance选项修改。
下载鼠标和图标主题，放置在/usr/share/icons文件夹下。

## 参考文献
1.https://linuxconfig.org/how-to-customize-dock-panel-on-ubuntu-18-04-bionic-beaver-linux
2.https://zhuanlan.zhihu.com/p/37852274
3.https://askubuntu.com/questions/15832/how-do-i-get-the-cpu-temperature
4.https://linuxhint.com/install_gnome3_extensions_ubuntu_1804/
5.https://linux.cn/article-9447-1.html
6.https://askubuntu.com/a/1062401
7.https://zhuanlan.zhihu.com/p/37852274
