---
title: linux vim
date: 2019-05-07 16:58:22
tags: 
 - linux
categories: linux
---


34.常用的vim命令
	
	0 $
	hjkl
	oO iI aA
	1G gg G nG
	n-space n-ENTER nG
	nj nk nh nl
	x dd yy p
	/word ?word
	n N
	:1,10s/word/word.rp/g(c)
	:1,$s/word/word.rp/g(c)
	ctrl+f ctrl+b
	J  
	u . ctrl+r

	:w [filename]
	:r [filename]
	:n1,n2 w [filename]


	:set nu

35.其他vim使用事项(290)
	
　　　中文编码
	tty1-tty6默认不支持中文编码　　
	修改终端　接口语系　
	LANG=zh_CN.big5


	ubuntu  don't have dos2UNIX or UNIX2dos  but is has tofrodos

	　　　	frodos filename
		todos filename
			-b .bak
			-v ver

	语系编码转换
	iconv -o保留原文件，-o加新文件名
	iconv -f big5 -t utf8 filename -o filename



