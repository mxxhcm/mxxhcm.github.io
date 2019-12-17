---
title: linux service and daemon
date: 2019-05-07 16:47:38
tags:
 - linux
categories: linux
---

## service和daemon
service（服务）：系统提供某些功能的一些服务(包括系统本身以及网络service)
daemon：实现service的程序叫做daemon

## daemon的分类
- stand alone daemon
- super daemon

### stand_alone daemon
独立启动，启动并加载到内存后就一直占用内存与系统资源运行。因此对于客户端的请求响应特别快。比如WWW的daemon(httpd)，FTP的daemon(vsftpd)

### super daemon
由一个统一daemon唤起的service，这个特殊的daemon叫做super daemon早期是inetd,后来被xinetd替代了。没有客户端请求时，service被关闭，收到客户端请求时，super daemon唤醒相应的service，请求结束后，这个service就会关闭，service反应时间会比较慢。常见的有telnetservice。
signal-control和interval-control，信号管理的daemon以及每隔一段时间主动执行某项job的daemon每一个service程序文件名都会加上d，d代表daemon。
    
## SysVInit service

### 配置文件路径
- /etc/rc.d/rcX.d/ (X 代表运行级别 0-6) # 不同runlevel的service存放位置
- /etc/rc.d/rc.local    # 用户自定义的service

### 自定义文件示例
创建/etc/init.d/shadowsocks_client service如下所示
``` txt
#!/bin/sh

### BEGIN INIT INFO
# Provides:          shadowsocks client
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: shadowsocks service
# Description:       shadowsocks service daemon
### END INIT INFO
start(){
　　  sslocal -c /etc/shadowsocks.json -d start
}
stop(){
　　  sslocal -c /etc/shadowsocks.json -d stop
}
case “$1” in
start)
　　　start
　　　;;
stop)
　　　stop
　　　;;
reload)
　　　stop
　　　start
　　　;;
\*)
　　　echo “Usage: $0 {start|reload|stop}”
　　　exit 1
　　　;;
esac
```

然后执行以下命令进行更新
~$:sudo chomod a+x /etc/init.d/shadowsocks_client
~$:sudo update_rc.d shadowsocks defaults

### 运行方式
service shadowsocks_client start

## SystemD
### 配置文件路径
- /etc/systemd/system	系统service，不要动。大部分是软连接，指向/usr/lib/systemd/sytem
- /run/systemd/system	Runtime units
- /usr/local/lib/systemd/system	管理员安装的System units
- /usr/lib/systemd/system	包管理器安装的System units(for centos)
- /lib/systemd/system   包管理器安装的System units(for debian/ubuntu)
- /etc/systemd/system/\*\*.service.wants/\*：此目录内的文件为链接文件，设置相依服务的链接。意思是启动了 \*\*.service 之后，最好再加上这目录下面建议的服务。
- /etc/systemd/system/vsftpd.service.requires/\*：此目录内的文件为链接文件，设置相依服务的链接。意思是在启动 vsftpd.service 之前，需要事先启动哪些服务的意思。

### 自定义unit文件示例
在/lib/systemd/system/创建ss_client.service，如下所示：
``` txt
[Unit]
Description=ss v4 client daemon
 
[Service]
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks_v4_client.json </dev/null &>>/home/mxxmhh/.log/ss-local.log 
WorkingDirectory=/home/mxxmhh/
# Restart=on-failure
StartLimitBurst=2
StartLimitInterval=30
User=mxxmhh
ExecReload=/bin/kill -SIGHUP $MAINPID
ExecStop=/bin/kill -SIGINT $MAINPID
 
[Install]
WantedBy=multi-user.target
```

然后执行以下命令
~$:sudo systemctl start ss-client.service
~$:sudo systemctl enable ss-client.service 
> Created symlink /etc/systemd/system/multi-user.target.wants/ss-client.service → /lib/systemd/system/ss-client.service.

执行以下命令发现报错
~$:sudo systemctl status ss-client.service
``` txt
ss-client.service: Start request repeated too
ss-client.service: Failed with result 'exit-c
```
根据参考文献13使用下列命令常看详细log
~$:journalctl -u ss-client.service
发现报错：
``` txt
ss-client.service: Failed at step USER spawning /usr/bin/sslocal: No such proces
```
然后根据参考文献12发现可能是自己的文件写的有问题，最后发现是user复制的时候出错了，修改之后就好了。执行以下命令加载修改后的配置文件，然后restart服务。
~$: sudo systemctl daemon-reload
~$: sudo systemctl restart ss-client.service
~$: sudo systemctl status ss-client.service

### Unit文件的编写
每个unit都有一个配置文件，定义了这个unit启动的条件。

#### Unit格式
下面是 SSH service的unit文件，service unit文件以.service 为文件名后缀。
~$:cat /etc/systemd/system/sshd.service
``` txt
[Unit]
Description=OpenSSH server daemon
[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStartPre=/usr/sbin/sshd-keygen
ExecStart=/usrsbin/sshd –D $OPTIONS
ExecReload=/bin/kill –HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target
```
Unit部分仅仅有一个描述信息;Service中，ExecStartPre定义启动service之前应该运行的命令；ExecStart定义启动service的具体命令行语法；Install部分，WangtedBy 表明这个service是在多用户模式下所需要的，multi-user.target 

#### Unit的配置文件区块
一个文件通常由[Unit]，[Service]（或者其他unit类型）和[Install]构成。
[Unit]区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。它的主要字段如下。
- Description：简短描述
- Documentation：文档地址
- Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
- Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
- BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
- Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
- After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
- Conflicts：这里指定的 Unit 不能与当前 Unit 同时运行
- Condition...：当前 Unit 运行必须满足的条件，否则不会运行
- Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败

[Install]通常是配置文件的最后一个区块，用来定义如何启动，以及是否开机启动。它的主要字段如下。
- WantedBy：它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中
- RequiredBy：它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
- Alias：当前 Unit 可用于启动的别名
- Also：当前 Unit 激活（enable）时，会被同时激活的其他 Unit

[Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下。
- Type：定义启动时的进程行为。它有以下几种值。
- Type=simple：默认值，执行ExecStart指定的命令，启动主进程
- Type=forking：以 fork 方式从父进程创建子进程，创建后父进程会立即退出
- Type=oneshot：一次性进程，Systemd 会等当前服务退出，再继续往下执行
- Type=dbus：当前服务通过D-Bus启动
- Type=notify：当前服务启动完毕，会通知Systemd，再继续往下执行
- Type=idle：若有其他任务执行完毕，当前服务才会运行
- ExecStart：启动当前服务的命令
- ExecStartPre：启动当前服务之前执行的命令
- ExecStartPost：启动当前服务之后执行的命令
- ExecReload：重启当前服务时执行的命令
- ExecStop：停止当前服务时执行的命令
- ExecStopPost：停止当其服务之后执行的命令
- RestartSec：自动重启当前服务间隔的秒数
- Restart：定义何种情况 Systemd 会自动重启当前服务，可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
- TimeoutSec：定义 Systemd 停止当前服务之前等待的秒数
- Environment：指定环境变量

### 日志
Systemd统一管理所有Unit的启动日志。带来的好处就是，可以只用journalctl一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是/etc/systemd/journald.conf。
``` shell
~$:sudo journalctl  # 查看所有日志（默认情况下 ，只保存本次启动的日志）
~$: sudo journalctl -k    # 查看内核日志（不显示应用日志）
# 查看系统本次启动的日志
~$:sudo journalctl -b    
~$:sudo journalctl -b -0
~$:sudo journalctl -b -1 # 查看上一次启动的日志（需更改设置）
# 查看指定时间的日志
~$:sudo journalctl --since="2012-10-30 18:17:16"
~$:sudo journalctl --since "20 min ago"
~$:sudo journalctl --since yesterday
~$:sudo journalctl --since "2015-01-10" --until "2015-01-11 03:00"
~$:sudo journalctl --since 09:00 --until "1 hour ago"
~$:sudo journalctl -n   # 显示尾部的最新10行日志
~$:sudo journalctl -n 20    # 显示尾部指定行数的日志
~$:sudo journalctl -f   # 实时滚动显示最新日志
~$:sudo journalctl /usr/lib/systemd/systemd # 查看指定服务的日志
~$:sudo journalctl _PID=1   # 查看指定进程的日志
~$:sudo journalctl /usr/bin/bash    # 查看某个路径的脚本的日志
~$:sudo journalctl _UID=33 --since today    # 查看指定用户的日志
# 查看某个 Unit 的日志
~$:sudo journalctl -u nginx.service
~$:sudo journalctl -u nginx.service --since today
~$:sudo journalctl -u nginx.service -f  # 实时滚动显示某个 Unit 的最新日志
~$:journalctl -u nginx.service -u php-fpm.service --since today # 合并显示多个 Unit 的日志
# 查看指定优先级（及其以上级别）的日志，共有8级
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug
~$:sudo journalctl -p err -b
~$:sudo journalctl --no-pager   # 日志默认分页输出，--no-pager 改为正常的标准输出
~$:sudo journalctl -b -u nginx.service -o json  # 以 JSON 格式（单行）输出
~$:sudo journalctl -b -u nginx.serviceqq -o json-pretty # 以 JSON 格式（多行）输出，可读性更好
~$:sudo journalctl --disk-usage # 显示日志占据的硬盘空间
~$:sudo journalctl --vacuum-size=1G # 指定日志文件占据的最大空间
~$:sudo journalctl --vacuum-time=1years # 指定日志文件保存多久
```

### systemctl 工具
~$:systemctl list-units     列出正在运行的 Unit
~$:systemctl list-units --all   列出所有Unit，包括没有找到配置文件的或者启动失败的
~$:systemctl list-units --all --state=inactive      列出所有没有运行的 Unit
~$:systemctl list-units --failed    列出所有加载失败的 Unit
~$:systemctl list-units --type=service  列出所有正在运行的、类型为 service 的 Unit
~$:systemctl list-unit-files    列出所有配置文件
~$:systemctl list-unit-files --type=service     列出指定类型的配置文件
~$:systemctl start foo.service	用来启动一个service (并不会重启现有的)
~$:systemctl stop foo.service	用来停止一个service (并不会重启现有的)。
~$:systemctl restart foo.service	用来停止并启动一个service。
~$:systemctl reload foo.service	当支持时，重新装载配置文件而不中断等待操作。
~$:systemctl condrestart foo.service	如果service正在运行那么重启它。
~$:systemctl status foo.service	汇报service是否正在运行。
~$:systemctl list-unit-files --type=service	用来列出可以启动或停止的service列表。
~$:systemctl enable foo.service	在下次启动时或满足其他触发条件时设置service为启用。创建一个符号链接从/etc/systemd/system/some_target.target.wants指向/lib/systemd/system或者/etc/systemd/system。
~$:systemctl disable foo.service	在下次启动时或满足其他触发条件时设置service为禁用
~$:systemctl is-enabled foo.service	用来检查一个service在当前环境下被配置为启用还是禁用。
~$:systemctl list-unit-files --type=service	输出在各个运行级别下service的启用和禁用情况
~$:systemctl daemon-reload	当您创建新service文件或者变更设置时使用。
~$:systemctl isolate multi-user.target (OR systemctl isolate runlevel3.target OR telinit 3)	改变至多用户运行级别。
~$:ls /etc/SystemD/system/\*.wants/foo.service	用来列出该service在哪些运行级别下启用和禁用。

## 配置文件
/etc/init.d/\*　# 基本上所有的service启动脚本都被放置在该目录
/etc/rcX.d/    # X指的是数字，从$0-6$，代表不同的run-level，是/etc/init.d/目录下service的软连接
/etc/systemd/system     # systemd的service文件位置，是/usr/lib/systemd/sytem的软连接
/etc/default    # 一些配置文件
/etc/\*  # 各service各自的配置文件

## service，initctl，systemctl命令对照表
Service 命令|UpStart initctl 命令|SystemD 命令|备注
---|---|
service foo start|initctl start|systemctl start foo.service	用来启动一个service (并不会重启现有的)
service foo stop|initctl stop|systemctl stop foo.service	用来停止一个service (并不会重启现有的)
service foo reload|systemctl reload foo.service	当支持时，重新装载配置文件而不中断等待操作。
service foo restart|initctl restart|systemctl restart foo.service	用来停止并启动一个service
service foo status|initctl status|systemctl status foo.service	汇报service是否正在运行。
service foo reload|initctl reload|systemctl reload foo.service	当支持时，重新装载配置文件而不中断等待操作。
service foo condrestart||systemctl condrestart foo.service	如果service正在运行那么重启它。

## xinted
xinted是/etc/init.d/目录中的一个脚本。
xinted 是inted的扩展，是super daemon，它本身管理了一系列的daemon，只有在用户调用时才由xinetd启动，他们要比独立的daemon启动晚。
!!!!xinted默认在ubuntu中是不存在的,
~$:sudo apt-get install xinetd
/etc/xinetd.conf   #super daemon配置文件
/etc/xinetd.d/\*    #它所管理的进程
/var/lib/\*  各service产生的数据库
/var/run/\*  各service的程序的pid记录处
    
### stand alone的启动
1. 用/etc/init.d/\*启动
~#:/etc/init.d/cron start|stop|status|restart|reload|force-reload
    
2.用service [service-name] (start|...)启动
service-name必须与/etc/init.d/相照应
     --status-all 将所有的stand_aloneservice列出来
~#:service --status-all
~#:service cron

### super daemon的启动方式
super daemon本身也是一个stand alone的service，但是它所管理的其他文件就不是了。
查看某个service是否可用。
~#:grep -i 'disable' /etc/xinted.d/\* # disable表示取消，若为yes，表示该service未开启，no表示开启
    
#### 示例
开启timeservice
~#:vim /etc/xinted.d/time
将disable改为no
重新启动xinted service
~#:service xinted restart
!!!!!注意是重启xinted service
    
查看该service的信息
~#:grep -i 'time' /etc/services
~#:netstat -nltp | grep 'time port'

### 默认值配置文件以及参数介绍
/etc/xinetd.conf
    log_type    SYSLOG daemon info 日志文件的记录service类型
    log_on_failure  发生错误时需要记录的信息
    log_on_success  成功启动时的记录信息
    cps 同一秒内的最大连接个数，若超过则暂停
    instance    同一service的最大连接数
    per_source  同一来源的客户端的最大连接数
    v6only  是否运行ipv6
    groups  
    umask
    
/etc/xinetd.d/
    service <service name>
    {
        disable 启动与否
        id  service识别
        server  程序文件名  这个service的启动程序
        server_args 程序参数    设置server_args=--daemon
        user    service所属id
        group   用户组  
        socket_type 数据包类型  stream|dgram|raw stream使用tcp,   udp使用dgram,raw代表erver需要与ip直接交互。
        protocol    数据包类型  tcp|udp与socket_type重复，
        wait    连接机制    yes(single) no(multi) 一般udp为yes，tcp为no
        instances   最大连接数
        per_source  单用户来源  (一个数字或者NULIMTED)
        cps 新连接限制
        log_type    日志文件类型    以什么日志选项记载和需要记载的等级(默认为info)
        log_on_success,log_on_failure,设置值,[PID,HOST,USERID,EXIT,DURATION]
            PID为service启动时的pid,host为远程主机的ip，userid为登陆者的账号，EXIT为离开时记录的项目，DURATION为该用户使用此service多久。
        env 额外环境变量设置    设置环境变量
        port    非正规端口号    设置不同的service与对应的端口号，port与service名必须与/etc/services的值相同
        redirect    service转址    [IP port] 将客户端的请求转到另一台主机
        includedir  调用外部设置    表示将某个目录所有文件都放入xinetd.conf中，
        bind    service端口锁定    运行此service的适配卡
        interface   与bind相同
        only_from   [0.0.0.0,192.168.1.0/24,hostname,domainname]设置为这里面的ip或者主机名才能访问，0.0.0.0表示所有主机皆能访问，如果是192.168.1.0/24则表示为C　class的域，即由(192.168.1.1~192.168.1.255)皆可登录。另外，也可选择域名，如bit.edu.cn表示运行北理工的ip登录你的主机
        no_acess    表示的是不可登录的主机
        acess_time  时间控制    [00:00-24:00,HH:MM-HH:MM]
        umask   设置用户新建目录或者文件时候的属性
    }

## 参考文献
1.《鸟哥的LINUX私房菜》
2.https://askubuntu.com/questions/911525/difference-between-systemctl-init-d-and-service
3.http://www.r9it.com/20180613/ubuntu-18.04-auto-start.html
4.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init1/index.html
5.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init2/index.html?ca=drs-
6.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/index.html?ca=drs-
7.http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html
8.https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/150.html
9.https://www.freedesktop.org/software/systemd/man/systemd.unit.html
10.https://unix.stackexchange.com/questions/206315/whats-the-difference-between-usr-lib-systemd-system-and-etc-systemd-system
11.https://stackoverflow.com/questions/35452591/start-request-repeated-too-quickly
12.https://superuser.com/questions/1156676/what-causes-systemd-failed-at-step-user-spawning-usr-sbin-opendkim-no-such-p
13.https://stackoverflow.com/questions/39202644/caddy-service-start-request-repeated-too-quickly
