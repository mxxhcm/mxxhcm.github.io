---
title: ubuntu 编译安装gcc
date: 2019-05-06 14:17:40
tags:
 - linux
 - gcc
categories: linux
mathjax: true
---


## 下载相应版本的安装包
国科大源：https://mirrors.ustc.edu.cn/gnu/gcc/
官网源：http://ftp.gnu.org/gnu/gcc/
我选择的是官方源，执行以下命令下载：
~\$:wget http://ftp.gnu.org/gnu/gcc/gcc-7.3.0.tar.gz


## 解压
~\$:tar xvf gcc-7.3.0.tar.gz
~\$:sudo cp -r gcc-7.3.0 /usr/local/src/
~\$:cd /usr/local/src/gcc-7.3.0/

## 创建安装目录
~\$:sudo mkdir /usr/local/gcc-7.3.0
~\$:sudo mkdir /usr/local/src/gcc-7.3.0/build
~\$:cd /usr/local/src/gcc-7.3.0/build

## 配置
~\$:sudo ../configure --prefix=/usr/local/gcc-7.3.0/ --enable-threads=posix --disable-multilib --enable-languages=c,c++
~\$:sudo make -j8
~\$:sudo make install


## 修改gcc版本
~\$:sudo update-alternativess --install /usr/bin/cc cc /usr/local/gcc-4.6.0/bin/gcc-4.6 30
~\$:sudo update-alternativess --install /usr/bin/c++ c++ /usr/local/gcc-4.6.0/bin/g++-4.6 30

~\$:sudo update-alternativess --config cc
