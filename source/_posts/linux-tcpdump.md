---
title: linux tcpdump
date: 2020-03-14 13:32:22
tags:
 - linux
categories: linux
---


## tcpdump
tcpdump是unix系统下一个命令行抓包工具，需要使用root权限。

## 常用参数
host，指定一台主机。
port，指定端口。
net，指定网段。
src,指定目标地址。
dst,指定源地址。
tcp/udp/arp等，指定通信协议

-i指定网络接口
-D列出所有的网口，linux网口命名(en开头的是以太网口，wl开头的是无线网卡）。
-S用绝对值列出TCP的序号。
-n使用数字名字而不是主机名字（比如说https的端口号使用443而不是https）。
-c num指定接收num个包。

## TCP包的一些字段
查看man手册。
Flag中
S是SYN，
F是FIN，
P是PUSH，（表示发送方通知接收方通讯层应该尽快的将这个报文段交给应用层。一般传输层都是隔几个报文统一上交数据。设置了PUSH就是尽快上交。）
R是RST，
U是URG，
W是ECN CWR
E是ECN-Echo
.是ACK。
win是对方接收缓冲区的大小。

## 查看DNS查询报文
在本机上运行`sudo tcpdump udp dst port 53`，即指定目标端口号为53即可。

## 查看TCP三次握手
使用命令`sudo tcpdump host www.baidu.com -c 10 -n -vv`可以查看TCP连接的前10个报文。


## 参考文献
1.https://blog.csdn.net/renrenhappy/article/details/5929702
2.https://segmentfault.com/a/1190000002554673
