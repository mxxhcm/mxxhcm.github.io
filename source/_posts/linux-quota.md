---
title: linux quota
date: 2019-05-07 16:53:04
tags:
 - linux
categories: linux
---


## quota
该命令显示磁盘使用情况和限额

## quota示范
文件系统开启quota
df -h /home
mount -o remount,usrquota,grpquota /home
mount | grep 'home'
cat /etc/mtab
/etc/mtab会被写入这些操作

或者写入/etc/fstab
vim /etc/fstab
LABEL=/home /home ext4 defaults,usrquota,groquota

umount /home
mount -a
mount | grep 'home'
  新建quota配置文件
sudo apt-get install quota
quotacheck [-avugmf]
	-c 创建磁盘配额数据文件
	-a 创建在/etc/mtab所有磁盘的配额数据库文件，使用此参数，后无需加			挂载点
	-u 创建用户的磁盘配额数据库文件
	-g 用户组的
	-m 把一起的磁盘配额信息清除，对/分区创建时，必须用此参数
	-v 显示创建的过程
  启动quota
quotaon [-avug]　[挂载点]
	-a 根据/etc/fstab的设置来启动有关的quota
	-v
  关闭quota
quotaoff [-aug] [mount-point]

  编辑quota
edquota [-ugtp] 
	-t 修改宽限时间 
	-p 复制范本，，模板账号为已存在并设置好quota的账号
		edquota -p 范本账号 -u 新账号
	edquota -u myquota
	edquota -g myquotagrp
  		edquota -t
	edquota -p quotagrp -u myquota

  quota的报表表
单一用户
quota [-ugvs]
	-s 以1024的整数倍显示

repquota [-a] [-uvgs] 
	/dev/sda[012]

　warnquota	root给用户以及root发邮件　P461

 在 /etc/warnquota.conf 中设置邮件内容

　setquota [-u|-g] name block(soft) block(hard) inode(soft) inode(hard) 文件系统		
setquota -u myquota5 2000 3000 0 0 /home
quota-uv myquota5

