---
title: lychee图床搭建
date: 2019-03-04 21:03:55
tags:
 - 图床
categories: 工具
---


## 安装
### 基本要求
1. web server (Apache, nginx, etc)
2. A MySQL database (MariaDB also works)
3. PHP 7.1 or later with the following extensions: session, exif, mbstring, gd, mysqli, json, zip, and optionally, imagick
~#:apt install nginx
~#:apt install mysql-server
~#:apt install php
### 配置nginx
重新安装nginx出现问题，见参考文献2。
### 配置mysql
~#:mysql -u root -p
默认密码是回车？？？
修改密码
~#:

### 配置php


## 参考文献
1.https://juejin.im/post/5c1b869b6fb9a049ad770424
2.https://segmentfault.com/a/1190000014027697?utm_source=tag-newest
