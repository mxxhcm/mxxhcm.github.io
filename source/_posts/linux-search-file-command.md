---
title: linux search file command
date: 2019-05-07 17:03:36
tags:
 - linux
categories: linux
---


17.命令与文件的查询
	
	a.脚本文件的查询which [-a] command 
	
	b.文件的查询
		whereis和locate 利用数据库查找
		find查找硬盘
		find可以查找具有特殊要求的文件，如查找文件所有者，文件大小,SUID
			等等

		find . -size -5k -a size +1k 是会把当前目录也列出来的
		
		whereis [-bmsu]
		-b 二进制文件
		-m manualz路径下的文件(说明文件)
		-s source源文件
		-u 不在上述范围的其他特殊文件
	
		locate  查找/var/lic/mlocate数据库内的数据，该数据库每天更新一次			可手动更新，updatedb,因为他是每天更新一次，所以可能会找				到已删除的文件或者是找不到新建立的文件。
		a.与时间有关的参数
			find /tmp mtime n/+n/-n
		b.与用户或者用户组有关的文件
			find / 	-uid n 
				-gid n
				-user name
				-group name
				-nouser
				-nogroup
		c.与文件权限或者名称有关的参数
			find / 	-name filename	
				-size [+-]SIZE
				-type TYPE[-fbcdls]
				-perm mode	刚好等于mode
				-perm -mode	全部包含
				-perm /mode	任意一个
		find / -perm +7000 -exec ls -l {} \;
			http://www.cnblogs.com/wanqieddy/archive/2011/06/09/2076785.html

	
