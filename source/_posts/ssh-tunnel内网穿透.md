---
title: ssh tunnel外网访问内网
date: 2019-06-10 21:33:16
tags:
 - linux
 - ssh
categories: linux
---

## ssh 命令介绍
ssh是一种安全传输协议，此外还有tunnel转发功能，可以用来内网渗透。
### 参数介绍
-L port:host:hostport，访问本机的port端口就相当于访问host的hostport端口。
将本机的某个端口转发到远端指定机器的指定端口。本机上分了一个socket监听port端口，一旦该端口有了连接，就通过一个ssh转发出去。
-R port:host:hostport，将远程主机的某个端口转发到指定的本地机器的指定端口。远程主机上分了一个socket监听port端口，一旦该端口有了连接，就通过一个ssh转发到指定的本地机器的指定端口。
-N 不指定脚本或者命令
-f 后台认证，需要和-N连用

-L和-R的区别，-L是ssh隧道，-R是ssh反向隧道。

## 示例
### 翻墙ssh -L
执行以下命令的本机(localhost)通过中间服务器(45.32.22.289)访问被屏蔽的网站(google)。
ssh -L 1234:google_ip:80 root@45.32.22.289
拿这个举个例子，可能不是很恰当，但是有助于理解。我自己的机器(A)是不能访问google(C)的，但是我有一台vps(B)，地址为45.32.22.289是可以访问google的，可以通过ssh隧道将A的端口(1234)通过B映射到C的端口(80)。

### 内网穿透ssh -R
外网A(123.123.123.123)访问处于内网B的(127.0.0.1)的机器。
ssh -N -f -R 2222:127.0.0.1:22 root@123.123.123.123

可以在外网机器A(123.123.123.123)上通过如下命令访问(-R)指定的内网机器B：
ssh -p 2222 userB@localhost
### 报错
Host key verification failed
直接把/home/username/.ssh/known_hosts中相应的给删了。

### 内网访问内网（挖洞）
A是家里的内网（无公网IP）上机器(196.)，B是VPS（有公网IP）(45.32.)，C是公司内网（无公网IP）机器(10.)。
要在家里的内网访问公司的内网，即A访问C。在C上建立ssh反向隧道：
ssh -N -f -R 2222:127.0.0.1:22 userB@B.ip
在A上访问：
ssh -p  2222 userC@B.ip

## 参考文献
1.https://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html
2.https://blog.creke.net/722.html
3.http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/
