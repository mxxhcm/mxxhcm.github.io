---
title: linux jobs nohup bg fg ...
date: 2019-05-07 16:19:29
tags:
 - linux
categories: linux
---

## nohup
nohup　[command parameters] [&] nohup不挂断地运行命令。
nohup命令忽略所有挂断（SIGHUP）信号，有&表示在后台执行，没有&表示在机前台执行，即使脱机或者注销系统后仍然会执行，输出为nohup.out

## &
在后台运行。
一般nohup和&会在一起使用。即nohup command &，表示在后台不挂断的执行command命令
STDOUT以及STDERR都会被显示在屏幕上，可以采用数据流重定向将其输入文件    
tar -cvj -f ~/my.bak/etc20161006.tar.bz2 /etc > ~/tmp/log.txt 2>&1 &
这样stdout以及stderr会被输入进~/tmp/log.txt

## jobs
jobs -l 查看运行的后台进程，当打开该进程的终端关闭时，就无法看到使用jobs查看该程序了。需要使用ps命令
jobs [-lsr] 查看目前后台的jobs
    -l 列出所有的后台jobs，包含pid
    -s 列出停止的后台jobs，
    -r 列出正在运行的jobs,

## fg, bg, ctrl+z
fg(foreground)将后台的工作拿到前台
    fg %jobnumber
    fg +/- [jobnumber]表示第几个后台工作，+表示最后一个被丢入后台，-表示最后第二个被丢入后台，最后第三个以及以上不显示
bg将程序放到后台
ctrl+z挂起程序，将正在工作的程序放入后台(避免被ctrl+c终止,而非系统的后台)

## 参考文献
1.https://baike.baidu.com/item/nohup/5683841?fr=aladdin
2.https://www.cnblogs.com/baby123/p/6477429.html
3.https://www.cnblogs.com/hf8051/p/4494735.html
