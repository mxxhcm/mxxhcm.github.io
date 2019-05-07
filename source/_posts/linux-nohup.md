---
title: linux nohup &
date: 2019-05-07 16:19:29
tags:
 - linux
categories: linux
---

## nohup
nohup不挂断地运行命令。
nohup命令忽略所有挂断（SIGHUP）信号，关闭终端之后，进程依然会继续运行。

## &
在后台运行。
一般nohup和&会在一起使用。即nohup command &，表示在后台不挂断的执行command命令

## 查看
### fg,bg,ctrl + z
fg将程序放到前台
bg将程序放到后台
ctrl+z挂起程序

### jobs
jobs -l 查看运行的后台进程，当打开该进程的终端关闭时，就无法看到使用jobs查看该程序了。需要使用ps命令

### ps
比如查看sslocal程序是否运行
ps aux | grep 'sslocal'

## 参考文献
1.https://baike.baidu.com/item/nohup/5683841?fr=aladdin
2.https://www.cnblogs.com/baby123/p/6477429.html
