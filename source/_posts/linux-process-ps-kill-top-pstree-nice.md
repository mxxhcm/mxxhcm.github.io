---
title: linux process ps kill top pstree nice
date: 2019-06-03 21:30:04
tags:
 - linux
 - shell command
categories: linux
mathjax: false
---

## 概述
这一节介绍和process相关的命令，包含ps,top,kill, pstree, nice, fuser, lsof, pidof, /proc/等

## ps查看进程
### 参数介绍
ps [-Aauf] [xlj]    
-A 所有的进程全部显示出来
a 现行终端机下所有程序，包含其他用户
u 有效用户相关的进程，主要以用户为主的格式来区分
f 用ASCII字符显示树状结构，表达进程间的关系
x　通常与a这个参数一块使用，显示所有程序，不以终端机来区分
l　较长，较详细的将该PID的信息列出
j　工作的格式

### 示例
~$:ps aux　查看系统所有的进程数据
~$:ps -lA　查看所有系统的数据
~$:ps axjf　连同部分进程树状态
~$:ps aux | grep 'sslocal' #查看sslocal程序是否运行
~$:ps ax # 显示当前系统进程的列表
~$:ps aux #显示当前系统进程详细列表以及进程用户
~$:ps -A  #列出进程号
~$:ps aux |grep 2222'|grep -v grep  # 找出所有包含2222的进程，grep -v 过滤掉含有grep字符的行

### aux 查看系统所有进程
~$:ps aux     # 使用BSD格式显示进程
输出
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
USER    
PID 
%CPU    该进程使用掉的CPU资源百分比
%MEM    该进程占用物理内存百分比
VSZ 该进程占用虚拟内存量
RSS 该进程占用固定内存量(KB)
TTY pts/0　表示由网络连接进主机的进程
STAT    进程状态
START   该进程被触发的时间
TIME    CPU时间
COMMAND     该进程实际命令

僵尸进程(<defunct>)

### ps -ef 
~\\$:ps -ef  # 使用标准格式显示进程
输出
UID        PID  PPID  C STIME TTY          TIME CMD
UID 用户名
PID 进程ID
PPID    父进程ID
C   CPU占用百分比
STIME   进程启动到现在的时间
TTY     在哪个终端上运行，ps/0表示网络连接
TIME    
CMD     命令的名称和参数

### -l仅查看自己相关的bash进程
~\\$:ps -l #仅查看自己相关的bash进程
输出
F S UID PID PPID C PR NI ADDZ SZ WCHAN TTY TIME CMD
F  说明进程权限
S　进程状态STAT
R(running)　S(sleep)　D(不可被唤醒的睡眠状态,通常是IO的进程)　T(stop)　Z(zombie僵尸状态)进程已终止，但无法被删除到到内存外,PCB还在，但是其他资源全部被收回，是由父进程负责收回资源。
UID/PID/PPID
C CPU使用率
PR/NI  Priority/Nice的缩写，此进程被CPU执行的优先级
ADDR/SZ/WCHAN都与内存有关，ADDR是kernel function ,指出该进程在内存的哪个部分，如果是个running的过程，显示-;SZ代表用掉多少内存;WCHAN表示目前进程是否运行，-表示正在运行
TTY 使用的终端接口
TIME    使用掉的CPU时间，而不是系统时间
CMD command缩写,造成此进程被触发的命令

## top 动态查看进程的变化
### 参数介绍
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
### 示例
~\\$:top -d 2 #每两秒钟刷新一次top，默认为5s
~\\$:top -b -n 2 > ~/tmp/top.out

### top输出内容
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
S   STAT
%CPU　CPU使用率
%MEM　内存使用率
TIME+  CPU使用时间的累加
COMMAND 

## pstree 查看进程树
### 参数介绍
pstree [-A|-U] [-up] 
    -A  各进程树直接以ASCII字符连接
    -U  各进程树之间以utf-8字符连接
    -u  显示进程所属账号名
    -p  显示pid
### 示例
~\\$:pstree -Aup

## kill管理进程
### 参数介绍
kill -signals %jobnumber 杀掉某个job    
    -l  列出所有signal
    -1  重新读取一次参数的配置文件
    -2  与ctrl+c　一样
    -9  强制删除一个job，非正常状态
    -15 让一个job正常结束
### 查看signal种类
~\\$:man 7 signal

### kill示例
kill -signal %jobnum
kill -signal pid    
这两种情况是不同的，第一种是job，第二种是pid,不能弄混

### killall将系统中所有以某个命令启动的服务全部删除
killall [-iIe]  用来删除某个服务
    -i iteractive
    -e exact    
    -I 忽略大小写

### killall示例
~\\$:killall utserver
~\\$:killall -1 syslogd
~\\$:killall -9 httpd
~\\$:killall -i -9 bash


## nice管理进程优先级
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
### 示例
~\\$:nice -n -5 vim &
    
### 已存在的进程调整nice值
renice [number] command
~\\$:ps -l | grep '\\*bash\\$'
~\\$:renice 10 \\$(ps -l|grep 'bash\\$' | awk '{print \\$4}')

## fuser找到使用某文件的程序
### 参数介绍
fuser [-muv] [-k [i] [-signal]] name
    -m  后面接的文件名会主动提到文件顶层
    -u  user
    -v  verbose
    -k  SIGKILL
    -i  询问用户，与-k搭配
    -signal  -1,-15等，默认为-9
### 示例
~\\$:mount -o loop ubuntu.iso /mnt/iso
~\\$:cd /mnt/iso
~\\$:umount /mnt/iso
error
~\\$:fuser -muv   /mnt/iso
.....
~\\$:cd 
~\\$:umount /mnt/iso

~\\$:fuser -muv /proc
~\\$:fuser -ki /bin/bash

## lsof找到被进程打开的文件
### 参数介绍
lsof [-uaU] [+d]
    -a 相当于and连接符
    -u 某个用户的相关进程打开的文件
    -U Unix like 的socket文件类型
    +d 某个目录下被打开的文件
### 示例
~\\$:lsof +d ~/Desktop
~\\$:lsof -u mxx | grep 'bash'
~\\$:lsof -u mxx -a -U

## pidof找出某个正在进行的进程的pid
### 参数介绍
pidof [-sx] program_name
-s 仅列出一个pid而不列出所有的pid
-x 列出该进程可能的ppid的pid

## 获取进程id
ps -A |grep "command" | awk '{print $1}'
pidof 'command'
pgrep 'command'

## /proc/\\* 文件
/proc/cmdline   加载kernel执行的参数
/proc/cpuinfo   CPU相关信息
/proc/devices   主要设备代号　与mknod相关
/proc/filesystems   目前已加载的文件系统
/proc/interrupts    系统上IRQ分配状态
/proc/ioports   目前系统上每个设备配置的I/O地址
/proc/loadavg   top/uptime显示的负载
/proc/kcore 内存大写
/proc/meminfo   free
/proc/modules   目前系统已加载的模块列表，可想成驱动程序
/proc/mounts　mount
/proc/swaps  系统加载的内存使用的分区记录
/proc/partitions　fdisk -l
/proc/pci　PCI总线上每个设备的详细情况　lspci
/proc/uptime　uptime
/proc/version　内核版本 uname -a
/proc/bus/\\*　总线设备，USB设备

## 具有SUID，SGID的程序
如passwd，当触发passwd之后，会取得一个新的进程与PID,该PID产生时通过SUID给予该PID特殊的权限设置。
在一个bash中执行passwd会衍生出一个passwd进程，而且权限为root
~\\$:passwd &
~\\$:pstree -up找到该进程

## 参考文献
1.《鸟哥的LINUX私房菜》基础篇
