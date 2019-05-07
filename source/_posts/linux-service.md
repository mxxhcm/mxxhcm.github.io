---
title: linux service
date: 2019-05-07 16:47:38
tags:
 - linux
categories: linux
---



服务(service):系统为了某些功能提供的一些服务(包括系统本身以及网络服务)
daemon:实现service的程序叫做daemon

	
99.daemon的分类
	a.stand_alone daemon
		自行独立启动，不用通过其他机制，启动并加载到内存后就一直占用内存与系统资源。因此对于客户端的请求响应特别快。
		WWW的daemon(httpd)，FTP的daemon(vsftpd)
		
	b.super daemon
		由一个统一的daemon来唤起服务，这个特殊的daemon叫做super daemon早期是inetd,后来被xinetd替代了。没有客户端请求时，服务被关闭，收到客户端请求时，super daemon唤醒相应的服务，请求结束后，这个服务就会关闭，服务反应时间会比较慢。常见的有telnet服务

	signal-control和interval-control
	信号管理的daemon以及每隔一段时间主动执行某项工作的daemon
	每一个服务程序文件名都会加上d，d代表daemon
	
100./etc/services服务端口的对应
	查询service的端口号
	服务以及端口号
	~#:grep 'time' /etc/services
	

101.daemon的启动方式以及启动脚本

　　a.一些配置文件
	/etc/init.d/*　 #基本上所有的服务启动脚本都被放置在该目录
	/etc/default	#一些配置文件
	/etc/*	#各服务各自的配置文件

	!!!!xinted默认在ubuntu中是不存在的,
	~$:sudo apt-get install xinetd
	/etc/xinetd.conf   #super daemon配置文件
	/etc/xinetd.d/*	   #它所管理的进程
	
	/var/lib/*	各服务产生的数据库
	/var/run/*	各服务的程序的pid记录处
	
    b.stand alone的启动
	用/etc/init.d/*启动
	~#:/etc/init.d/cron start|stop|status|restart|reload|force-reload
	
	用service [service-name] (start|...)启动

	service-name必须与/etc/init.d/相照应
	--status-all 将所有的stand_alone服务列出来
	
	~#:service --status-all
	~#:service cron
	
	grep -i 'disable' /etc/xined.d/*
　　c.super daemon的启动方式
	super daemon本身也是一个stand　alone的服务，但是它所管理的其他文件就不是这样了
	
	查看某个服务是否可用。
	~#:grep -i 'disable' /etc/xinted.d/*
	disable表示取消，若为yes，表示该服务未开启，no表示开启
	
	若要开启time服务
	~#:vim /etc/xinted.d/time
	将disable改为no
	重新启动xinted服务
	~#:service xinted restart
	!!!!!注意是重启xinted服务
	
	查看该服务的信息
	~#:grep -i 'time' /etc/services
	~#:netstat -nltp | grep 'time port'
	

　　 d.默认值配置文件以及一些参数的值
	/etc/xinetd.conf
	log_type	SYSLOG daemon info 日志文件的记录服务类型
	log_on_failure	发生错误时需要记录的信息
	log_on_success	成功启动时的记录信息
	cps	同一秒内的最大连接个数，若超过则暂停
	instance	同一服务的最大连接数
	per_source	同一来源的客户端的最大连接数
	v6only	是否运行ipv6
	groups	
	umask
	
	service <service name>
	{
		
	}
	
	disable	启动与否
	id	服务识别
	server	程序文件名	这个服务的启动程序
	server_args	程序参数	设置server_args=--daemon
	user	服务所属id
	group	用户组	
	socket_type	数据包类型	stream|dgram|raw stream使用tcp,   udp使用dgram,raw代表erver需要与ip直接交互。
	protocol	数据包类型	tcp|udp与socket_type重复，
	wait	连接机制	yes(single) no(multi) 一般udp为yes，tcp为no
	instances	最大连接数
	per_source	单用户来源	(一个数字或者NULIMTED)
	cps	新连接限制
	log_type	日志文件类型	以什么日志选项记载和需要记载的等级(默认为info)
	log_on_success,log_on_failure,设置值,[PID,HOST,USERID,EXIT,DURATION]
		PID为服务启动时的pid,host为远程主机的ip，userid为登陆者的账号，EXIT为离开时记录的项目，DURATION为该用户使用此服务多久。
	env	额外环境变量设置	设置环境变量
	port	非正规端口号	设置不同的服务与对应的端口号，port与服务名必须与/etc/services的值相同
	redirect	服务转址	[IP port] 将客户端的请求转到另一台主机
	includedir	调用外部设置	表示将某个目录所有文件都放入xinetd.conf中，
	bind	服务端口锁定	运行此服务的适配卡
	interface	与bind相同
	only_from	[0.0.0.0,192.168.1.0/24,hostname,domainname]设置为这里面的ip或者主机名才能访问，0.0.0.0表示所有主机皆能访问，如果是192.168.1.0/24则表示为C　class的域，即由(192.168.1.1~192.168.1.255)皆可登录。另外，也可选择域名，如bit.edu.cn表示运行北理工的ip登录你的主机
	no_acess	表示的是不可登录的主机
	acess_time	时间控制	[00:00-24:00,HH:MM-HH:MM]
	umask	设置用户新建目录或者文件时候的属性

	e.端口号小于1024启动者身份一定要是root
	
	f.实际动手设置一个/etc/xinted.d/下的服务

