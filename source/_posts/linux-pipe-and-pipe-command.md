---
title: linux pipe and pipe command
date: 2019-06-17 19:59:30
tags:
 - linux
categories: linux
---

## PIPE
管道命令仅会处理stdout并不会处理stderrout，管道命令必须要能接受前一个命令传回来的数据成为stdinput

## head tail

## cut
cut -d '分隔字符' -f (fields)fields为数字
cut -c 字符范围

	cut -d ':' -f 2,3
	echo $PATH | cut -d ':' -f 2,4

	cut -c 20-30
	export | cut -c 12-

	不过cut 对于多个空格当做分隔字符的处理做的不够好

## grep
grep [-aincv] [--color=auto] '关键字' filename
	-a 将binary文件以text的方式查找数据
	-i 忽略大小写
	-c 计算查找到的字符串的个数
	-n 顺便输出行号
	-v 反向选择
### grep -n  '^$' regular_express
grep '^the' file
grep '[^[:lower:]]' file
grep '\.$' file
grep '^[^a-zA-Z]' file
grep 'go\{2,3\}g' file

对比
ls -l /etc/a*
grep -n '^a.*/' 


## dmesg
dmesg 查看内核信息
dmesg | grep -n A3 B2 'eth'
	A --after  B --before

## last
last | grep 'mxx' | cut -d '' -f 1

## sort,wc,uniq
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

## uniq
uniq 去重
	[-il] [file]
	-i 不区分大小写
	-c 进行计数	
	
	last | cut -d ' ' -f 1 | sort | uniq
	首先需要排序　才能去重	
last | cut -d ' ' -f 1 | sort | uniq -c

## tee
tee双重重定向将一份数据可以同时传到文件内以及屏幕中
	last | tee last.list | sort 

## tr
tr 删除一段文字或者对文字内容进行替换(如删除dos中的换行符^M)
	[-ds]
	-d 删除信息中的某个字串
	-s 替换重复字符

	last | tr '[a-z]'  '[A-Z]'
	echo $PATH -d ':/'
	cat /root/passwd | tr -d '\r' > passwd.linux
	
## col
col 简单处理
	[-xb]
	-x 将tab键换成空格键
	
	cat  manpath.config | col -x | cat -A | more
	

## join
join 将两个文件中具有相同数据的一行相加
	join [-ti12] file1 file2
	-i 大小写忽视
	join -t ':' passwd shadow
	join -t ':' -1 4 passwd -2 3 group

## paste
paste直接将两行粘在一起，默认并以tab键分开
	-d后面可以加分隔字符默认以tab分隔
	-表示来自standard input的数据的意思
	paste shadow passwd
	cat shadow | paste passwd - | head -n 3

## expand
expand将tab键换成空格默认是8个空格
	-t 参数可以自行设定空格数
	nl file | expand -t 6 - | cat -A

## split
split [-bl] file PREFIX
	-b后面加文件欲切割成的文件大小
	-l以行数来切割
	split -b 1M /etc/termcap termcap
	ls -l termcap*

	cat termcap* >> termcapback
	
	ls -l / | spilt -l 10 -lsroot
	wc -l lsroot

## xargs
xargs产生某个命令的参数
	[-pne0]
	-p 执行每个命令询问用户
	-e 是EOF的意思，后面可接一个字符，当xargs遇到这个字符，便会停止				操作	
	-n 后面接次数，每次command命令执行时，要使用几个参数
-用来代替stdout以及stdin
tar -cvf - /home | tar xvf -	



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

## 参考文献
1. 《鸟哥的LINUX私房菜》
