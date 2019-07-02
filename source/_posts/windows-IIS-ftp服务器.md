---
title: windows IIS ftp服务器
date: 2019-06-09 10:45:35
tags:
 - ftp
 - windows
categories: 工具
---


## 在服务端搭建ftp服务器
### 创建ftp目录
这里我选择在D盘，创建目录D://Ftp作为我的ftp文件存放目录
## 添加ftp用户（可选）
其实这里的ftp用户就是windows 操作系统的用户，为了安全起见，我们不选我们工作时登录的用户，选择重新创建一个新的用户。这里添加了新用户ftpuser作为ftp登录用户。

### 开启IIS服务
打开控制面板下的程序，选择启用或关闭Windows功能，选中Internet Information Services中的FTP服务器和Web管理工具（如下图）。等待其加载完所有组件

### 设置ftp站点
执行完第3步以后，打开计算机管理，在服务和应用程序下我们就可以看到Internet Infromation Service，点击它，右击网站选择新建一个ftp站点，配置见下图，完成以后启动该ftp。

## 防火墙允许通过
打开 控制面板>系统和安全>Windows防火墙>允许的应用
点击允许其他应用，这时添加windows服务主进程的路径"C:\Windows\System32\svchost.exe"，这时候防火墙就允许ftp访问通过了。

## 在客户机登录ftp服务器
在资源管理器中输入ftp://ip地址，输入账号密码后即可成功访问ftp服务器。


## 参考文献

