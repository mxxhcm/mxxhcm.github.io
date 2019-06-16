---
title: linux search file command
date: 2019-05-07 17:03:36
tags:
 - linux
categories: linux
---

## 命令与文件的查询
### 文件查询
- find
- whereis
- locate

whereis和locate利用数据库查找，find查找硬盘。

### 命令查询
- which 

## find
find从硬盘中查找文件，还可以查找具有特殊要求的文件，如查找文件所有者，文件大小,SUID等等

### 与时间有关的参数
~$:find /tmp mtime n/+n/-n

### 与用户或者用户组有关的文件
find / 	-uid n 
    -gid n
    -user name
    -group name
    -nouser
    -nogroup

### 与文件权限或者名称有关的参数
find / 	-name filename	
    -size [+-]SIZE
    -type TYPE[-fbcdls]
    -perm mode	刚好等于mode
    -perm -mode	全部包含
    -perm /mode	任意一个

### find示例
``` shell
# 查找/home/maddpg目录下所有__pycache__目录和文件
find /home/maddpg -name **__pycache__ 
# 查看根目录下所有权限为7000的文件
find / -perm +7000 -exec ls -l {} \;
# 查找当前目录下size在1k到5k之间的文件，+表示大于，-表示小于
find . -size -5k -a -size +1k # 是会把当前目录也列出来的
```

## whereis
### 参数介绍
whereis [-bmsu]
    -b 二进制文件
    -m manualz路径下的文件(说明文件)
    -s source源文件
    -u 不在上述范围的其他特殊文件

## locate
locate  查找/var/lic/mlocate数据库内的数据，该数据库每天更新一次可手动更新，updatedb,因为他是每天更新一次，所以可能会找到已删除的文件或者是找不到新建立的文件。

## which 
### 参数介绍
which -a command 列出所有的位置。
which command 列出第一个找到的位置

### 示例
~$:which -a python
> /home/mxxmhh/anaconda3/bin/python
/usr/bin/python

~$:which pip
> /home/mxxmhh/anaconda3/bin/pip

## 参考文献
1.《鸟哥的LINUX私房菜》
2.http://www.cnblogs.com/wanqieddy/archive/2011/06/09/2076785.html
