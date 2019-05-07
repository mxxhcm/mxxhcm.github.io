---
title: linux at cron anacron
date: 2019-05-07 16:50:33
tags: 
 -linux
categories: linux
---


例行性工作
78.at仅执行一次的工作调度
	/etc/at.allow
	/etc/at.deny
	at [-lmdvc] TIME
		-m 当at完成时，即使没有输出信息，以mail通知用户
		-v 可以使用较明显的时间格式列出at调度中的工作
		-c 可以列出后面接的该项工作的实际命令内容
		-d 相当于atrm，可以取消一个at工作
		-l 相当于atq，列出目前系统上所有该用户的at调度
	TIME
	  HH:MM	
		04:00
	  HH:MM YYYY-MM-DD
	  	05:00 2016-10-05
	  HH:MM[pm|am] [Month] [Date] 
		04 January 10
	  HH:MM [am|pm] + number [minutes|hours|days|weeks]
		now + 5 minutes
		05pm + 3days
	at 04pm + 10 days
	at -c number
	at -l//atq
	
	batch //当空闲时刻再执行

79.例行性工作调度
	cron
	ubuntu中没有下面两个配置项
	/etc/cron.allow
	/etc/cron.deny
	默认为所有用户都可以使用crontab
	crontab [-u user] [-ler]
		-u　只有root能设置这个参数
		-l　列出所有的crontab工作内容
		-e  编辑crontab的内容
		-r　删除所有crontab的内容
	
	新建crontab
	！！！！周与月日不可共存
	~#:crontab -e
	* * * * * cmd
	分钟　小时　日期　月份　星期
	* 任何时刻
	- 时间范围 0-59
	, 分隔 3,6,9
	/n */5每过五个单位(分钟，小时，天)
	!!!crontab -r
	~#:crontab -r # 会把所有的crontab都删处理
	要删除一个，请用crontab -e
	
　　　开启/var/log/cron.log
	~#:vim /etc/rsyslog.d/50-default.conf
	将rsylog文件中的#cron.*前的#去掉
	~#:service rsyslog restart
	~#:service cron restart
	~#:vim /var/log/cron.log
	
	/var/spool/cron/crontabs/
	该目录下为不同账号的crontab内容
	
	/etc/crontab 为系统的例行性任务
	run-parts
	/etc/cron.daily
	/etc/cron.hourly
	/etc/cron.monthly
	/etc/cron.weekly
	
	自己可以新建run-parts，周期性执行
	*/2 * * * * run-parts /etc/cron.minutely
	*/5 * * * * run-parts /root/runcron
	或者直接执行命令
	* * * * * mxx /home/mxx/outputtime_fiveminutes.sh

80.anacron 处理非24小时开机的系统
	
	anacron [-usfn] [job]
		-u 更新记录文件的时间戳
		-s 开始连续执行各项job，依据记录文件的时间戳判断是否进行
		-f 强制执行，不管时间戳
		-n 立即进行未进行的任务，而不延迟

	/etc/cron*/*ana*
	/etc/cron.daily/0anacron
	0表示最先被执行，让时间戳先被更新，避免anacron误判

	/etc/anacron	anacron的设置
	
　	/var/spool/anacron/*	
	记录最近一次执行anacron的时间戳


	
