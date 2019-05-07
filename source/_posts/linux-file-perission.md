---
title: linux file perission
date: 2019-05-07 17:08:26
tags:
 - linux
categories: linux
---


8.rwx   -r   读权限	查询文件名数据
	-w   写权限	a.新建新的文件与目录 b.删除文件或者目录 c.重命名以及转移文件或者目录
	
	-x   可执行权限	a.进入某目录 b.切换到该目录（cd命令）
		cd - 回到上一次工作的目录	
	!!!能不能进入某一目录只与该目录的x权限有关，如果不拥有某目录的x权限，即
	使拥有r权限，那么也无法执行该目录下的任何命令
	
	但是即使拥有了x权限，但是没有r权限，能进入该目录但是不能打开该目录，因为	没有读取的权限。
	
	
	sudo mkdir -p /test/test
	-R(递归的将某目录下所有的文件以及目录全部修改
	sudo chown -R root:root test	

	改变文件或者目录的权限
	a. sudo chmod 777 test
	b. sudo chmod +x test
	c. u--user  g--group  o--others  a--all
	   sudo chmod u=rwx,g=r,o=r test
	改变文件或者目录的属主
	a. sudo chown root:root test
	b. sudo chown root test	
	改变属组
	sudo chgrp root test

9.	/bin:可以被单用户执行的命令。其下的命令可以被root用户和普通用户执行，如cat,cd,cp,date,chown,
		chmod,等等
	/sbin/:开机过程所需要的，只能被root用户所执行，普通用户只能进行查询，包括与开机，还原系统所需要
		的命令
	/usr/bin:绝大部分的用户可使用命令都在这里
	/usr/sbin/:服务器所需要的某些软件程序
	/usr/local/sbin:本机自行安装的软件产生的系统执行文件
	
	/	根目录
	/etc	系统的配置文件
	/lib	执行文件所需要的函数库与内核所需要的模块
	/bin	重要执行文件
	/sbin	重要的系统执行文件
	/dev	所需要的设备文件
	这五个目录必须和根目录放在一块。
10.根目录最好小一些，将一些经常用到的文件目录(/home:/usr:/var:/tmp与根目录分到不同的分区	
　　　	因为越大的分区，放入的数据也就越多，出错的几率也就越大，而如果根目录出现问题，系统就可能会出现
	问题。

11.一次性全部输出
	cat 	-n --打印行号(包含空白行)
       		-b --打印行号(不含空白行)
　     		-a --将一些看不见的字符用特殊符号打出来　空格用"^I^" 回车用"$"
   	tac(反向列出)  cat --> tac
   	nl(添加行号打印)
    分屏输出
	more	
	less
	head 	[-n number]
	tail
    非纯文本文件(二进制文件)
	od -t 	c	ASCII
		a	默认字符
		o	octal 八进制
		d	decimal 十进制
		f	浮点数
		x	十六进制
12.修改文件时间
	stat filename	#列出该文件的各种时间
	
	touch 	-a	仅修改访问时间
		-t	后面接欲修改的时间而不用当前时间，格式为[YYMMNNhhmm]
		-d	接欲修改的日趋而不用当前日期，也可以用--date="时间或者日	期"	
		-c	仅修改文件的时间(文件状态改变的时间)
		-m	仅修改mtime(文件内容被更改的时间)

	-d 和　-t 修改的是mtime和atime 但是不能修改　ctime
	touch 	-d "2 days ago" testtouch
	touch 	-t 150929 testtouch
	
	ls -l 默认显示的是mtime,是内容修改的时间(modify)
	touch --time=ctime　　ctime 显示的是状态被改变的时间,指的是文件属性和权限发生改变。
	touch --time=atime    atime 访问时间显示的是最近文件被访问的时间(acess),cat and more可以,但是像ls和stat不会改变
	
	ls -lc   #chagne state
	ls -lu   #acess time访问时间
	ls -l	 #modify time 
   mkdir 
	mkdir -p /dir/my_dir
	mkdir -p /dir/{test,test1,test2}
13.umask

	用户创建文件时一般不应有执行的权限
	所以创建文件的默认权限为666也即　-rw-rw-rw-
	但是目录需要有执行的权限，应为777,即-rwxrwxrwx
	
	~$:umask
	0002
	~$:umask -S
	u=rwx,g=rwx,o=rx

	第一个与特殊权限有关，后三个与一般权限有关，在创建文件或者目录时，会将um	-ask所对应的权限拿掉
	即新建文件时:
	(-rw-rw-rw-)-(--------w-) = (-rw-rw-r--)所以创建文件的一般权限为-rw-rw-r	--
	同理可得创建目录时的权限应该为drwxrwxr-x即775
	
	要修改umask的值，可直接在输入umask后接所要减去的权限
	
	即
	~$:umask 002

	一般情况下root成员的umask为022，这是为了安全考虑，但是一般成员的umask是0	22,即保留了同用户组的写入权利

如果同一个用户组的不同用户无法修改另一个用户的文件，那么就可能是同组成员的创建文件时的默认权限不同，可以用umask 俩修改

14.chattr与lsattr 设置文件的隐藏属性。

	change attributes
	chattr -i 设置文件不可以被删除(包括root用户)
	       -a 设置文件只能增加数据，而不能删除或者修改文件(如登陆文件)
	
	chattr	+i +a 可以增加文件的隐藏属性，其他属性不变
		-i -a 可以除去文件的隐藏属性，其他属性不变
		=i a  仅有=后面的属性
	
	lsattr 查看文件的隐藏属性

15.文件特殊权限　在文件或者目录中除了rwx外，还会出现s,t,S,T权限
	
	a.SUID	当s出现在文件所有者的x权限上时，
	
	如
	~$:ls -l /usr/bin/passwd
	-rwsr-xr-x 1 root root ....
	~$:ls -l /etc/shadow
	-rw-r----- 1 root shadow 
	
	用户密码存在/etc/shadow内，当用户想要修改密码时，可以使用passwd进行修改
	用户mxx对于/etc/shadow没有任何权限，但是对于/usr/bin/passwd拥有r-x权限，	所以可以执行passwd命令，由于在passwd命令中有SUID权限，所以mxx在执行pass-	wd命令时，会暂时获得passwd拥有者即root的权限，所以接下来可以用passwd修改	/etc/shadow
	
	！！！此权限仅可用于二进制程序中，且仅在执行该程序的过程中有效，此外只对	于文件有效，对于目录也是无效的

	
	b.SGID　当s权限出现在用户组的x权限时
	
	如
	~$:ls -l /usr/bin/mlocate /var/lib/mlocate/mlocate.db
	-rwx--s--x 1 root mlocate
	同SUID类似，程序执行者会获得该程序用户组的支持，
	
	还可以用在目录上
	若该用户在此目录下具有w权限，用户创建的新文件的用户组与此目录组的用户组		相同
	~$:su root	
	~#:mkdir test
	~#:ls -l test
	drwxrwxr-x 2 root root ...
	~#:chmod 2777 test
	~#:ls -l test
	drwxrwsrwx 2 root root
	~#:su mxx
	~$:cd test
	~$:touch test
	-rw-rw-r-- 1 mxx root ....
	！！！此权限对于目录以及文件都有效

	
	c.SBIT 当t出现在others的x权限上时
	
	~$:ls -l /tmp
	drwxrwxrwt 13 root root ....

	用户对于某个目录具有wx的权限，即可以写入的权限，相当于说目录的属主给了用	户属组或者其他人的身份，并拥有w的权限，那么也就是说这个用户具有删除属主
	创建的文件或者目录的删除等权限。但是如果该目录拥有了SBIT的权限，那么该用	户就只能删除自己所创建的文件，而不能删除属主所创建的文件。

	
	！！！此权限只针对目录有效
	
　　如何设置文件以及目录的特殊权限(SUID 4,SGID 2 ,SBIT 1)
	
	最前面的一位为文件的特殊权限
	直接用chmod 4755 filename  就可以了
	还可以通过加法来实现，如SUID为u+s,SGID为g+s,SBIT为o+t
	
	此外还有大写的S和T，代表空
	如
	~$:chmod 7666 test 
	~$:ls -l test
	-rwSrwSrwT 1 mxx mxx 
	因为s和t都是替代x的，而当文件所有者以及其他用户用户组都没有x的时候，所以	就不用说其他的操作了，所以也就为空了


16.file 查看文件类型　　data或者ASCII或者binary
	file /usr/bin/passwd

