---
title: ubuntu 终端快速访问某个目录
date: 2019-04-15 18:49:57
tags:
 - ubuntu
categories: 工具
mathjax: true
---

## 动机
在写博客的过程中，每次在终端中进入该目录，都要输好长的命令，在想着有没有什么简单的方法。后来就在网上找到了。

## 方法
利用alias命令进行重命名
这里给出一个具体的例子，我的博客文件存放在/home/mxxmhh/github/blog/source/_posts下，
在/home/mxxmhh/.bashrc文件中添加如下一行即可(当然也可以在其他配置文件中添加)：
alias posts='cd /home/mxxmhh/github/blog/source/_posts'
然后执行
~\$:source /home/mxxmhh/.bashrc
即可。
接下来可在终端输入
~\$:posts
直接访问该目录。

## 参考文献
1.https://www.cnblogs.com/wlsphper/p/6782625.html
