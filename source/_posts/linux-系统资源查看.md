---
title: linux 系统资源查看
date: 2019-06-03 21:20:52
tags:
 - linux
categories: linux
mathjax: true
---

## 查看内存
free [-bkmg] -t
    -t  输出结果中显示free和swap加在一起的总量
free -h 

## 查看磁盘
df -h 查看磁盘占用情况
du -h --max-depth=2 统计当前目录下深度为2的文件和目录大小

## 查看cpu占用
top

## 查看系统与内核相关信息
uname [-asrmpi]
    -a 所有系统相关的信息，
    -s 系统内核名称
    -r 内核版本
    -m 本系统硬件名称
    -p CPU类型，与-m类似，显示的是CPU类型
    -i 硬件平台

## 查看系统启动时间与工作负载    
uptime

## 查看网络
netstat [-atunlp]
    -a 将目前系统上所有连接监听，socket列出来
    -t 列出tcp网络数据包的数据
    -u 列出udp网络数据包的数据
    -n 不列出进程的服务名称，以端口号来显示
    -l 列出目前正在网络监听(listen)的服务
    -p 列出该网络服务的进程pid
~\$:netstat -tnlp　　#找出目前系统上已经在监听的网络连接及其PID

## 分析内核产生的信息
dmesg | more
dmesg | grep -i 'sd'
dmesg | grep -i 'eth'

## 动态分析系统资源使用情况
man vmstat 查看各字段含义

vmstat      动态查看系统资源
vmstat [-a]     display active and inactive memory
vmstat [-d] report sisk statistics
vmstat [-p 分区]　Detailed statistics about partition
vmstat [-S 单位]  B K M G 
vmstat [-f]  displays the num of forks since boot
~\$:vmstat 
~\$:vmstat -a 
~\$:vmstat -p /dev/sda1
~\$:vmstat -S M
~\$:vmstat -f

