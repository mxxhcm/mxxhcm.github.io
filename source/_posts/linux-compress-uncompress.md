---
title: linux compress uncompress
date: 2019-05-07 17:01:24
tags:
 - linux
categories: linux
---


29.gzip ,zcat ,bzip2,bzcat , gunzip ,bunzip2,7z,zip,rar

	gzip 和 bzip2　公用	
		-d 解压缩参数	
		-# #取1-9　压缩等级 -1最快
		-v 显示压缩比
		-k 保留原文件
		-z 压缩参数
		-c 将压缩过程中产生的数据输出到屏幕上(压缩后的数据)可以将其输出重定向
		-t 检验一个压缩文件的一致性

	zip [-dmbrfFg]
		-d　从zip文件中移除一个文件
		-m  将特定文件移入zip文件，且删除特定文件
		-g　将文件压缩附加到zip文件中
		-r  包括子目录
		-f  以新文件取代旧文件
		-F　修复已经损毁的压缩文件
		-b　暂存文件的路径
		-v　显示详细信息
		-u　值更新改变过的文件
		-T　测试zip文件是否正常
		-x　不需要压缩的文件
	~#:zip -r myfile.zip ./*
	~#:zip -d myfile.zip myfile　 //删除压缩文件内的某个文件
	~#:zip -g myfile.zip myfile   //向一个压缩文件内添加新文件
	~#:zip -u myzip
	
	unzip [-dnovj]
		-v  查看压缩文件目录，但是不解压
		-d  指定解压到的目录
		-n　不覆盖已有文件
		-o　覆盖已有文件
		-j  不重建文档的目录结构，把所有文件解压到同一目录下
	
	7z
	~#:apt-get install p7zip
	7z [x|a] [-rotr]
		a　代表添加文件到压缩包
		x　代表解压缩文件
		-r 表示递归所有文件
		-t 制定压缩类型
		-o 指定解压到的目录
	~#:7z a -t 7z -r myfile.7z  ~/myfile
	~#:7z x myfile.7z -r -o ~/

	rar
	~#:apt-get install rar
	~#:rar x myfile.rar
	~#:rar a myfile.rar myfile	
30.tar命令
	
	-c  --create
	-x  --extract
	-t  --list
	-v  --verbose
	-f  --file
	-j  --bzip2
	-z  --gzip --gunzip 
	-C  --directory DIR
           	change to directory DIR
	
	-p  --preserve-permissions, --same-permissions
	-P  --absolute-names

	--exclude=file

	~#:tar -cvj -f ~/my_bak/etc.newer.passwd.tar.bz2 --newer-mtime="2016/09/23"			/etc/*
	
	~#:tar -cvj -f ~/my_bak/etc.tar.gz   /etc
	~#:tar -xvj -f ~/my_bak/etc.tar.gz -C /tmp
	~#:tar -xvj -f ~/my_bak/etc.tar.gz |etc/shadow
	~#:tar -tfj -f ~/my_bak/etc.tar.gz | grep 'shadow'

	tarfiel tarball
	
	磁带机/dev/st0
	tar -cv -f /dev/st0 /home /root /etc

31. dump 与　restore

	dump 
		-S	size
		-u	update    recode the dump time to /var/lib/dumpdates
			-u只能对level 0 操作
		-j	add compress bz2(默认压缩等级为 2)
		-v	verbose   详细的
		-W	列出/etc/fstab中的具有dump设置的分区是否被备份过
		-f	
		-level	备份的等级(0-9) 对于文件系统有九个等级
					对于单个目录只有0级

	~#:dump -0u -f /root/etc.dump /etc
	
	
	restore
		-t	list
		-C	compare
		-i	itera
		-r	r	

	~#:restore -t -f /root/boot.dump
	~#:restore -C -f /root/boot.dump
	~#:mkdir test_restore
	~#:sudo fdisk /dev/sdb
		new a /dev/sdb2
	~#:cd test_restore	
	~#:restore -r -f /root/boot.dump
	~#:restore -i -f /root/etc.dump.bz2
		help
		ls
		add
		delete
		extract
		quit

32.mkisofs
	-o +生成的镜像名
	-r 记录更完整的文件信息，包括UID,GID与权限等
	-v 显示构建iso的过程
	-V 新建Volume
	-m exclude 排除某文件
	-graft-point 
	
	~#:mkisofs -r -v -o ~/my.iso/system.iso -m /home/lost+found -graft-point/home=/home /root=/root /etc=/etc

33.dd 
	可以备份整块硬盘或者整块磁盘　　包括　superblocks以及boot sector 等等

