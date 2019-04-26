---
title: hexo 安装
date: 2019-04-26 18:56:46
tags:
 - hexo 
categories: 工具
mathjax: true
---

## 安装
### 安装git
~$:sudo apt install git
### 安装nodejs
#### ubuntu 16.04安装
注意在ubuntu 16.04的时候，安装的nodejs版本太老了。一直报错，
>
在官网下载linux 64位nodejs安装包
解压之后放在/usr/local/nodejs目录下。
然后在PATH环境变量中添加/usr/local/nodejs/bin即可（在.bashrc文件中修改即可）。
使用以下命令查看nodejs版本
~$node -v
#### ubuntu 18.04安装
在ubuntu 18.04可以直接使用以下命令安装。
~$:sudo apt install nodejs
~$:sudo apt install npm

### 安装hexo 
~$:sudo npm install -g hexo-cli

## 配置
创建文件夹
~$:git clone your repo
或者直接
~$:hexo init your repo
安装依赖包
~$:npm install 
