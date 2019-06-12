---
title: linux bash
date: 2019-05-07 16:55:17
tags:
 - linux
categories: linux
mathjax: true
---


## type 查看命令的来自于哪里　　
是bash还是外部命令还是别名
file外部命令
alias别名
builtin内置在bash内
	-t -p -a

type -t ls 
~$:alias 以file builtin alias 列出该命令的类型
type -a ls 列出所有的名为ls的命令

## echo 与 unset
~\$:echo $PATH
~\$:echo ${PATH}

""内的特殊字符可以保持原有特性　var="lang is $LANG" 那么 echo $var 输出			var=en_US.UTF-8
''内的特殊字符仅保存为一般文本

反单引号`可以获得其他命令的信息
version=`uname -r`  =$(uname -r)	

## env以及export查看常见变量
/etc/profile
/etc/bash.bashrc

RANDOM产生0~32767的随机数
产生0-9的用declare -i number=$RANDOM*10/32767    echo $number


HOME
SHELL
HISTSIZE
MAIL
PATH	/etc/environment
LANG
RANDOM

　　set查所有变量

HISTFILE=~/.bash_history
MAILCHECK
PS1	提示符的设置
$	关于本shell的PID
?	上个变量的回传码，正确返回0，错误返回其他值，可以利用代码差错
OSTYPE HOSTTYPE MACHTYPE	主机硬件与内核的等级	


    export将自定义变量转换为环境变量
    
    locale -a 文件的语系

    read 赌球来自键盘输入的变量
-p	用户可以输入提示语
-t	光标等待用户输入时间
    
~\$:read -p "hello" -t 10 variable

    declare 声明变量的类型　　默认为字符串
-x	声明环境变量 
-i	将变量定义为整形
-a	将变量定义为数组
-r	将变量设置为readonly  若要删除该变量，必须退出该bash重进
-p	单独列出变量的类型

    ulimit 与文件系统以及程序的限制关系
-a 后面不接任何参数,可以列出所有的限制额度
-c 某些进程发生错误，系统可能会将该进程在内存中的信息写成文件，这种文件			就称为内核文件(core file)。此为限制每个内核文件的最大容量
-f 此文件可以创建的最大文件容量,一般为2G:
-d 进程可使用的最大断裂内存(segment)容量
-l 可用于锁定(lock)的内存量
-t 可使用的最大CPU时间
-u 用户可使用的最大进程(process)数量

-H hard limit 严格的限制　　必须不能超过
-S soft limit 警告的限制　　可以超过，但要有警告信息

## 变量的使用
　　变量内容的测试与内容替换
    echo ${variable#*}
    echo ${variable##*}

    echo ${variable%*}
    echo ${variable%%*}

    echo ${variable/bin/BIN}
    echo ${variable//bin/BIN}

    变量的测试与替换

    new_var=${old_var-content}
    用新的变量的值区替代旧的变量的值，新旧变量可为同一个，若old_var不存在，则将
content的值给new_var,而若old_var的值存在则将其赋给new_var;
　　加上:的话，即使old_var为空的话，也会用content的值去赋给new_var

    username=""
    username=${username:-root}
    echo $username
    
    将-换成=是将原变量一同更改
　　将-换成?是当变量不存在时，可以发出错误信息

    
## 命令别名设置功能　

alias
alias lm='ls -al'

unalias
## history命令与文件
history (n)列出最近的第(n)条命令

!number	执行history的第number条命令
!command　由最近的命令开始搜寻开头为command的命令
!!	执行上一个命令

## Bash Shell的操作环境

　　　路径与命令查找顺序
先由相对路径或者绝对路径寻找
a.alias
b.builtin
c.$PATH这个变量的顺序找到的第一个命令
　　　
　　  bash的登陆界面以及欢迎信息
/etc/issue  #
/etc/issue.net  #提供telnet远程登陆，当使用telnet连接到主机时显示该内容
/etc/motd(?)->/etc/update-motd.d/　
/etc/issue	\d \l \m \n \o \r \t \s \v
	\d 本地端时间的日期
	\l 显示第几个终端机
	\m 显示硬件等级
	\n 显示主机的网络名称
	\o 显示domain name
	\r 操作系统的版本
	\t 显示本地端时间的时间
	\s 操作系统的名称
	\v 操作系统的版本

      bash的环境配置文件	

login shell 以及non-login shell
   /etc/profile系统整体的设置
   ~/.profile用户个人设置

login shell
    /etc/profile
	/etc/inputrc	/etc/profile.d/*sh
    ~/.profile
	~/.bashrc	/etc/bashrc
    开始操作bash

non-login shell
	取得non-login shell 时，该bash配置文件仅会读取~/.bashrc
	
	
source 配置文件名
如
	source ~/.bashrc
	. ~/.bashrc

/etc/manpath.config使用man时man page的路径到哪里去找
用tarball的方式安装的时候,那么man page可能放置在/usr/local/softpackage/\	man里，需要以手动的方式将该路径加入到/etc/man.config里面

     终端机的环境设置
stty　　setting tty(终端机的意思)
	-a 将所有的stty参数列出来
    
    如何设置呢  比如将erase设置为ctrl+h来控制stty erase ^h
    ctrl + c 终止目前的命令
　　ctrl + d 输入结束，例如邮件结束
　　ctrl + m ENTER
    ctrl + u 在提示符下，将整行命令删除
　　ctrl + z 暂停目前的命令

set
　set $-　那个$-变量内容是set的所有设置 
  uvxhHmBC
    	
/etc/inputrc其他的按键设置功能

     通配符与特殊符号
通配符* ? [] - ^
特殊字符　# \ | ; ~ $ & ! / >,>> <,<< '' "" ``或者$() () {}

## 数据流重定向

> >>
>　输出重定向　
	ls -ld /etc/ ~/directiry/etc_dir
std output 
	find / -name ~/.bashrc > find_result 
std error output
	find / -name ~/.bashrc >fing_right 2> find_error !必须为2>
	
	find / -name ~/.bashrc > find_result 2>/dev/null
	
	find / -name ~/.bashrc >find_result 2>&1
>> 追加到某个文件结尾

< <<
cat > catfile
>
>
> ctrl + D
从文件中读入数据
cat > catfile < ~/.bashrc
重定义输入结束符
cat > catfile << "eof"
 	
&& || 命令从左到右依次执行　根据回传码$0的值，继续向右执行命令

## PIPE

管道命令仅会处理stdout并不会处理stderrout
管道命令必须要能接受前一个命令传回来的数据成为stdinput

cut -d '分隔字符' -f (fields)fields为数字
cut -c 字符范围

	cut -d ':' -f 2,3
	echo $PATH | cut -d ':' -f 2,4

	cut -c 20-30
	export | cut -c 12-

	不过cut 对于多个空格当做分隔字符的处理做的不够好

	last最近登录的用户

grep [-aincv] [--color=auto] '关键字' filename
	-a 将binary文件以text的方式查找数据
	-i 忽略大小写
	-c 计算查找到的字符串的个数
	-n 顺便输出行号
	-v 反向选择
dmesg | grep -n A3 B2 'eth'
	A --after  B --before

last | grep 'mxx' | cut -d '' -f 1

sort wc uniq
	
sort 	排序
	[-fbMnrutk] [file]
		-f 不区分大小写
		-u uniq
		-t 分隔符
		-k 以第几个字段进行排序
		-n 以数字进行排序(默认是以字母)
		-m 反向排序
	eg:
	sort -t ':' -k 3 -n /etc/passwd	
	or
	cat /etc/passwd | sort -t ':' -k 3 -n

uniq 去重
	[-il] [file]
	-i 不区分大小写
	-c 进行计数	
	
	last | cut -d ' ' -f 1 | sort | uniq
	首先需要排序　才能去重	
last | cut -d ' ' -f 1 | sort | uniq -c

tee双重重定向将一份数据可以同时传到文件内以及屏幕中
	last | tee last.list | sort 

tr 删除一段文字或者对文字内容进行替换(如删除dos中的换行符^M)
	[-ds]
	-d 删除信息中的某个字串
	-s 替换重复字符

	last | tr '[a-z]'  '[A-Z]'
	echo $PATH -d ':/'
	cat /root/passwd | tr -d '\r' > passwd.linux
	
col 简单处理
	[-xb]
	-x 将tab键换成空格键
	
	cat  manpath.config | col -x | cat -A | more
	

		
join 将两个文件中具有相同数据的一行相加
	join [-ti12] file1 file2
	-i 大小写忽视
	join -t ':' passwd shadow
	join -t ':' -1 4 passwd -2 3 group


paste直接将两行粘在一起，默认并以tab键分开
	-d后面可以加分隔字符默认以tab分隔
	-表示来自standard input的数据的意思
	paste shadow passwd
	cat shadow | paste passwd - | head -n 3

expand将tab键换成空格默认是8个空格
	-t 参数可以自行设定空格数
	nl file | expand -t 6 - | cat -A

split [-bl] file PREFIX
	-b后面加文件欲切割成的文件大小
	-l以行数来切割
	split -b 1M /etc/termcap termcap
	ls -l termcap*

	cat termcap* >> termcapback
	
	ls -l / | spilt -l 10 -lsroot
	wc -l lsroot

xargs产生某个命令的参数
	[-pne0]
	-p 执行每个命令询问用户
	-e 是EOF的意思，后面可接一个字符，当xargs遇到这个字符，便会停止				操作	
	-n 后面接次数，每次command命令执行时，要使用几个参数
-用来代替stdout以及stdin
tar -cvf - /home | tar xvf -	

正则表达式

## dmesg 查看内核信息

## grep -n  '^$' regular_express
grep '^the' file
grep '[^[:lower:]]' file
grep '\.$' file
grep '^[^a-zA-Z]' file
grep 'go\{2,3\}g' file

对比
ls -l /etc/a*
grep -n '^a.*/' 

nl 打印出文件并加上行号

## sed 工具	

[-in]
-i	直接修改文件内容
-n	静默
-e 直接在shell下编辑
-c replace
-a append
-p print

nl file | sed '2,3d'
nl file | sed '$a add a test'
nl file | sed -n '5,7p'
nl file | sed '2,5c jkadfk\
			>fdasf\
			>asfddf '
nl file | sed 's/s_place/s_replace/g'	
nl file | sed '/^$/d'
## egrep 扩展正则表达式

egrep -n '^$|^#' file
egrep -n 'go?d' file　0个或者一个?之前的字符
egrep -n 'go+d' file　一个及以上+之前的字符
grep -n 'go*d' file　0个或者0个以上*之前的字符
## printf 格式化打印

printf '%s\t %s\t %s\t \n' $(cat file)
printf '%10s %5i %5i \n' $(cat file)
## awk

last | awk '{print $1 "\t" S3 "\t" $4 NF NR}'
cat /etc/passwd | awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t" $3 }'

cat /etc/passwd | awk 'NR==1{printf "%10s %10s %10s %10s ",$1,$2,$3,"tot		al"}
	NR>=2{total=$2+$3;
		printf "%10s %10d %10d %## f",$1,$2,$3,total"}'
## diff 文本比较，通常比较一个文件的不同版本	
diff [-bBi] file1 file2
	-b　忽略一行中仅有多个空白的区别
	-B  忽略空白行的区别
	-i  忽略大小写
diff test.old test.new
diff -Naur test.olc test.new > test.patch
patch 补丁
cat test.patch
patch -pN test.patch 更新旧版
patch -R -pN test.patch 恢复为旧版

学习shell script
看一下自己写的/home/mxx/scripts/delete_dir
## echo $(($num1 operand $num2))	
进行运算
## source file.sh   sh file.sh   ./file.sh
source 是将该shell拿到父进程中来执行，所以各项操作都会在该bash内执行
sh和./是开启一个新的shell来执行
## test
test [-rwxfd]
	[-nt -ot -ef ]
	[-eq -nq -gt -lt -ge -le]
	[-z ]
	[-a -o]

test -r filename
test "$filename" == "content"

[ "$filename" == "$varible" ]

## $# $@ $* 
$#:变量个数
$@:变量内容
$*:

## netstat -tuln 列出本机上的网络服务

## if,case,function,
     if
if [ "$filename" == "mxx" ];
then
	...
elif [ ... ];
then
	....
else
	....
fi
     case
case $variable in
	"first")
	...
	;;
	"second")
	...
	;;	
     function
function printfinfo()
{
	echo "This is a file"
}
## 循环 while, until , for
      while	
while [ "$i" -lt "$n" ]
do
	..
done
      until
until [ "$i" -ge "$n" ]
do
	...
done

      for
for var in f1 f2 f3...
do
	...
done

      for ((初始化值;控制值;执行步长))


## seq 产生一系列数
seq [-s]
	
~$:seq -s " " 3 10
3 4 5 6 7 8 9 10

## sh [-vxn] my.sh
sh -x	执行过程
sh -n	查询语法问题

## id和finger
id 用来显示某个用户的id信息
finger 用来分析某个用户信息


