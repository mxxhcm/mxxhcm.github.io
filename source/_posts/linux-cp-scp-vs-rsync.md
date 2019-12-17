---
title: linux cp, scp vs rsync
date: 2019-06-16 16:29:17
tags:
 - linux
 - shell command
categories: linux
---

## cp
### 参数介绍
### 示例

## scp
### 参数介绍
### 示例

## rsync
rsync相对于scp有以下优点：
1.支持断点续传
2.支持ssh
3.可分块传输
~$:rsync options source destination

### 参数介绍
rsync [-zvra] source destination 
    -z  传输前进行压缩
    -v  显示详细信息
    -r  递归拷贝
    -a  保留时间戳，owner,group
    -e ssh  使用ssh
    --partial    单个文件的断点续传
    --progress  显示同步进度
    -P等于--paritial和--progress一同使用
    --include   只同步某些目录
    --exclude   不同步某些目录

### 示例
~$:rsync -avc --exclude=\*\*__pycache__  --exclude=\*\*data --exclude=\*\*tmp --exclude=\*\*result_pictures experimental/ ~/
~$:rsync -rP source_dir target_dir

## 参考文献
1.https://stackoverflow.com/questions/4585929/how-to-use-cp-command-to-exclude-a-specific-directory
2.https://www.cnblogs.com/bangerlee/archive/2013/04/07/3003243.html
