---
title: linux process nohup, & ...
date: 2019-05-07 16:19:29
tags:
 - linux
categories: linux
---

## nohup
nohup不挂断地运行命令。
nohup命令忽略所有挂断（SIGHUP）信号，关闭终端之后，进程依然会继续运行。

## &
在后台运行。
一般nohup和&会在一起使用。即nohup command &，表示在后台不挂断的执行command命令

## 查看
### fg,bg,ctrl + z
fg将程序放到前台
bg将程序放到后台
ctrl+z挂起程序

### jobs
jobs -l 查看运行的后台进程，当打开该进程的终端关闭时，就无法看到使用jobs查看该程序了。需要使用ps命令

### ps
比如查看sslocal程序是否运行
ps aux | grep 'sslocal'

### 查看进程
ps ax # 显示当前系统进程的列表
ps aux #显示当前系统进程详细列表以及进程用户
ps -A  #列出进程号

### 获取进程id
ps -A |grep "command" | awk '{print $1}'
pidof 'command'
pgrep 'command'


81.job control
	tar -cvj -f ~/my.bak/etc20161006.tar.bz2 /etc &
	&表示将程序放入后台
	但是如果有STDOUT以及STDERR都会被显示在屏幕上，可以采用数据流重定向来完成	
	tar -cvj -f ~/my.bak/etc20161006.tar.bz2 /etc > ~/tmp/log.txt 2>&1 &
	这样stdout以及stderr会被输入进~/tmp/log.txt
	
82.将正在工作的程序放入后台(避免被ctrl+c终止,而非系统的后台)
	ctrl + z 
	[num] 表示第几个后台工作，+表示最后一个被丢入后台，-表示最后第二个被丢入后台，最后第三个以及以上不显示
	
	jobs [-lsr] 查看目前后台的jobs
		-l 列出所有的后台jobs，包含pid
		-s 列出停止的后台jobs，
		-r 列出正在运行的jobs,
	
83.fg (foreground)将后台的工作拿到前台
	fg %jobnumber
	fg +/-

   bg 让后台的工作变成运行状态

84.kill -signals %jobnumber 删除某个job
	-l	列出所有能使用的signal
	-1	重新读取一次参数的配置文件
	-2	与ctrl + c　一样
	-9	强制删除一个job，非正常状态
	-15	让一个job正常结束
	
	
85.nohup　[命令与参数] [&]有&表示在终端机后台执行，没有&表示在终端机前台执行，
	即使脱机或者注销系统后仍然会执行
	输出为nohup.out
	
	
86.进程的查看
　　　a.
	ps [-Aauf] [xlj]	
		-A 所有的进程全部显示出来
		a 现行终端机下所有程序，包含其他用户
		u 有效用户相关的进程，主要以用户为主的格式来区分
		f 用ASCII字符显示树状结构，表达进程间的关系
		x　通常与a这个参数一块使用，显示所有程序，不以终端机来区分
		l　较长，较详细的将该PID的信息列出
		j　工作的格式
	~#:ps aux　查看系统所有的进程数据
	~#:ps -lA　查看所有系统的数据
	~#:ps axjf　连同部分进程树状态
    b.
	~#:ps -l #仅查看自己相关的bash进程
	
	F S UID PID PPID C PR NI ADDZ SZ WCHAN TTY TIME CMD
	F  说明进程权限
	S　进程状态STAT
		R(running)　S(sleep)　D(不可被唤醒的睡眠状态,通常是IO的进程)　T(stop)　Z(zombie僵尸状态)进程已终止，但无法被删除到到内存外,PCB还在，但是其他资源全部被收回，是由父进程负责收回资源。
	UID/PID/PPID
	C CPU使用率
	PR/NI  Priority/Nice的缩写，此进程被CPU执行的优先级
	ADDR/SZ/WCHAN都与内存有关，ADDR是kernel function ,指出该进程在内存的哪个部分，如果是个running的过程，显示-;SZ代表用掉多少内存;WCHAN表示目前进程是否运行，-表示正在运行
	TTY	使用的终端接口
	TIME	使用掉的CPU时间，而不是系统时间
	CMD	command缩写,造成此进程被触发的命令
	
    c.	
	~#:ps aux 查看系统所有进程
	
	USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
	USER	
	PID	
	%CPU	该进程使用掉的CPU资源百分比
	%MEM	该进程占用物理内存百分比
	VSZ	该进程占用虚拟内存量
	RSS	该进程占用固定内存量(KB)
	TTY	pts/0　表示由网络连接进主机的进程
	STAT	进程状态
	START	该进程被触发的时间
	TIME	CPU时间
	COMMAND		该进程实际命令
	
	僵尸进程(<defunct>)

    d.top动态查看进程的变化
	top [-d 数字] | top [-bnp]
		-d　整个进程界面更新的秒数
		-b  以批次的方式执行top，
		-n  与-b搭配，意义是，需要进行几次top的输出结果
		-p  指定某个PID来查看
		在top执行中可以使用的命令
			?查询所有的命令
			P 按CPU使用率排序
			M Mem排序
			N PID排序
			T CPU累计时间排序
			k 给某个PID一个信号
			r 给某个PID重新制定一个nice值
			q 离开top软件
	~#:top -d 2 #每两秒钟刷新一次top，默认为5s
	~#:top -b -n 2 > ~/tmp/top.out
	
    e.top显示的内容
	第一行top
	目前时间　
	开机到现在时间　up n days , hh:mm 
	登陆的用户
	系统在1,5,15分钟时的负载，batch工作方式负载小于0.8即为这个负载，代表的是1,5,15分钟，系统平均要负责多少个程序。越小说明系统越闲
	第二行task
	目前进程总量，分别有多少个处于什么状态，不能有处于zombie的进程，
	第三行%cpu
	wa代表的是I/Owait，系统变慢都是由于I/O产生问题较多
	第四五行内存和swap使用情况，swap被使用的应该尽量少，否则说明物理内存实在不足。
	第六行
	PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
	PID　每个进程ID
	USER　该进程所属用户
	PR　Priority 进程的优先级顺序，越小越早被执行
	NI　Nice 与Priority有关，越小越早被执行
	VIRT
	RES
	SHR
	S	STAT
	%CPU　CPU使用率
	%MEM　内存使用率
	TIME+  CPU使用时间的累加
	COMMAND 

   g.pstree [-A|-U] [-up]  #进程树
		-A  各进程树直接以ASCII字符连接
		-U  各进程树之间以utf-8字符连接
		-u  显示进程所属账号名
		-p  显示pid
	~$:pstree -Aup


87.进程的管理
	查看signal种类请使用
		man 7 signal
     a.
	kill -signal PID
		kill -signal %jobnum
		kill -signal pid	
	这两种情况是不同的，第一种是job，第二种是pid,不能弄混

     b.killall [-iIe]  用来删除某个服务，利用killall可以将系统中所有以某个命令启动的服务全部删除
	-i iteractive
	-e exact	
	-I 忽略大小写
	
	~#:killall utserver
	~#:killall -1 syslogd
	~#:killall -9 httpd
	~#:killall -i -9 bash
	

88.进程的调度
	PRI(priority)与NI(nice)
	PRI值是由内核动态调整的，用户无法直接调整PRI值
	PRI(new)=PRI(old)+nice
	
	nice值虽然可以影响PRI，但是并不是说原来PRI为50,nice为5,就会让PRI变为55,
这是需要经过系统分析之后决定的
	
	nice值
	a.可调整范围为-20~19
	b.root可随意调整任何人的nice值-20~19间的任意一个值	
	c.一般用户仅可以调整自己nice值，且范围在0~19
	d.一般用户仅可将nice值调高，而无法降低
	e.调整nice值的方法
	    新执行的命令手动设置nice值
		nice -n [number] command
		~#:nice -n -5 vim &
	    
       	    已存在的进程调整nice值
		renice [number] command
		~#:ps -l | grep '*bash$'
		~#:renice 10 $(ps -l|grep 'bash$' | awk '{print $4}')

88.系统资源的查看

	a.查看内存使用情况
		free [-bkmg] -t
		-t	输出结果中显示free和swap加在一起的总量
		
	b.查看系统与内核相关信息
		uname [-asrmpi]
			-a 所有系统相关的信息，
			-s 系统内核名称
			-r 内核版本
			-m 本系统硬件名称
			-p CPU类型，与-m类似，显示的是CPU类型
			-i 硬件平台
	c.查看系统启动时间与工作负载	
		uptime
	
	d.查看网络
		netstat [-atunlp]
			-a 将目前系统上所有连接监听，socket列出来
			-t 列出tcp网络数据包的数据
			-u 列出udp网络数据包的数据
			-n 不列出进程的服务名称，以端口号来显示
			-l 列出目前正在网络监听(listen)的服务
			-p 列出该网络服务的进程pid
		~#:netstat -tnlp　　#找出目前系统上已经在监听的网络连接及其PID
	e.分析内核产生的信息
		dmesg | more
		dmesg | grep -i 'sd'
		dmesg | grep -i 'eth'
	f.动态分析系统资源使用情况
		man vmstat 查看各字段含义

		vmstat		动态查看系统资源
		vmstat [-a]     display active and inactive memory
		vmstat [-d]	report sisk statistics
		vmstat [-p 分区]　Detailed statistics about partition
		vmstat [-S 单位]  B K M G 
		vmstat [-f]  displays the num of forks since boot
		~#:vmstat 
		~#:vmstat -a 
		~#:vmstat -p /dev/sda1
		~#:vmstat -S M
		~#:vmstat -f
89.特殊文件与程序
	
	a.具有SUID，SGID的程序
		如passwd，当触发passwd之后，会取得一个新的进程与PID,该PID产生时通过SUID给予该PID特殊的权限设置。
	在一个bash中执行passwd会衍生出一个passwd进程，而且权限为root
	~#:passwd &
	~#:pstree -up找到该进程

90./proc/* 的意义
	/proc/cmdline	加载kernel执行的参数
	/proc/cpuinfo	CPU相关信息
	/proc/devices	主要设备代号　与mknod相关
	/proc/filesystems	目前已加载的文件系统
	/proc/interrupts	系统上IRQ分配状态
	/proc/ioports	目前系统上每个设备配置的I/O地址
	/proc/loadavg	top/uptime显示的负载
	/proc/kcore	内存大写
	/proc/meminfo	free
	/proc/modules	目前系统已加载的模块列表，可想成驱动程序
	/proc/mounts　mount
	/proc/swaps  系统加载的内存使用的分区记录
	/proc/partitions　fdisk -l
	/proc/pci　PCI总线上每个设备的详细情况　lspci
	/proc/uptime　uptime
	/proc/version　内核版本 uname -a
	/proc/bus/*　总线设备，USB设备

91 查询已打开文件或者已执行程序打开的文件
　　a.fuser [-muv] [-k [i] [-signal]] name
	 #通过文件找到使用该文件的程序
		-m	后面接的文件名会主动提到文件顶层
		-u	user
		-v	verbose
		-k	SIGKILL
		-i	询问用户，与-k搭配
		-signal  -1,-15等，默认为-9
	
	~#:mount -o loop ubuntu.iso /mnt/iso
	~#:cd /mnt/iso
	~#:umount /mnt/iso
	error
	~#:fuser -muv 	/mnt/iso
	.....
	~#:cd 
	~#:umount /mnt/iso

	~#:fuser -muv /proc
	~#:fuser -ki /bin/bash

　　　b.lsof [-uaU] [+d]
	#找到被进程打开的文件
		-a 相当于and连接符
		-u 某个用户的相关进程打开的文件
		-U Unix like 的socket文件类型
		+d 某个目录下被打开的文件

	~#:lsof +d ~/Desktop
	~#:lsof -u mxx | grep 'bash'
	~#:lsof -u mxx -a -U

      c.pidof [-sx] program_name
	#找出某个正在进行的进程的pid
		-s 仅列出一个pid而不列出所有的pid
		-x 列出该进程可能的ppid的pid

## 参考文献
1.https://baike.baidu.com/item/nohup/5683841?fr=aladdin
2.https://www.cnblogs.com/baby123/p/6477429.html
3.https://www.cnblogs.com/hf8051/p/4494735.html
