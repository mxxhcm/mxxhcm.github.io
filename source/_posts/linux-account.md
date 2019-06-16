---
title: linux account
date: 2019-05-07 16:54:11
tags:
 - linux
categories: linux
---

## 账户配置文件
- /etc/passwd 
- /etc/shadow 
- /etc/group 
- /etc/gshadow

### /etc/passwd
mxx:x:1000:1000:mxx,,,:/home/mxx:/bin/bash
账号名称,密码，UID,GID,用户信息说明列，主文件夹，shell

### /etc/shadow
mxx:........:17034:0:99999:7:::
账号名称，密码，最近密码更改日期，密码更改多久后才能重新更改，密码多长时间需要重新修改，密码需要修改前多少天发送警告，密码过期后宽限时间，账号失效日期，(形式和密码更改日期一样)，保留字段

### /etc/group
用户组名称，用户组密码,GID,此用户组支持的账号名称

### /etc/gshawod
用户组名，
密码列，开头为!表示无合法密码
用户组管理员的账号
该用户组的所属账号

UID | 用户
--|--
0   |  系统管理员
1-99    |   系统账户
100-499 |   用户创建的系统账号
500-65535   |  一般用户

## 修改密码
一般账户:passwd
root账户:重启后进入单用户维护模式
忘记密码后，以各种方式清空/etc/shadow中root的密码字段。登陆后再用passwd修改密码 

## uesr管理
usermod -G group user将一个用户加入其他用户组
初始用户组用户的/etc/passwd的第四个字段即为该用户的初始用户组的GID
groups查看当前登陆用户的用户组。第一个为有效用户组
newgrp更改用户的有效用户组，但是用户组必须当前用户支持的用户组
UID/GID密码参数的设置在 /etc/login.defs

### useradd添加用户
#### 参数介绍
useradd [-ugGmMcdrsef]　　调用/etc/default/useradd的数据
    -u UID          /etc/skel用户主文件加参考基准目录
    -g initial group  
    -G 这个账户可以加入的其他用户组
    -m 创建用户主文件　
    -M 不创建用户主文件
    -s 接一个默认shell
    -r 创建一个系统账户 
    -c /etc/passwd的第五列说明  
    -d 制定某个目录成为主文件夹
    -e 后面跟一个日期YYYYMMDD写入shadow的第八字段，账号的失效日期从1970年来总日数，若账号失效，无论密码是否正确，都无法登陆
    -f 后面接shadow的第七字段,判定密码是否会失效,0为立即失效,-1为永不失效(密码只会过期强制登陆时重新设置),大于0的表示如n，如果在n天后，没有登陆修改密码，那么在n天后密码会失效，再也无法登陆，但是在如果在n天内登陆并修改密码，就可以继续使用。
    -D useradd的默认值  

## 例子
~$:useradd -d /home/mxxhcm -k /etc/skel/ -m mxxhcm -s /bin/bash

### passwd修改密码
#### 参数介绍
passwd [账号]　[--stdin] -[luSnxwi]
    -l lock
    -u unlock
    -S 密码相关参数
    -n next　多长时间不能修改第四个字段
    -x 多少天必须修改　第五个字段
    -w warn第六个字段
    -i 失效日期　第七个字段

#### 示例
~#:passwd　后面没有接密码，就是修改当前用户的密码
~#:echo "passwd" | passwd --stdin user

### change修改user信息
#### 参数介绍
chage [-ldEImMW] 账号名
    -l　列出详细参数
    -d　第三字段
    -E　第八字段　账号失效　
    -I　第七字段　密码失效
    -m　第四字段
    -M　第五字段
    -W　第六字段

#### 示例
chage -d 0 user

### user信息修改
#### 参数介绍
usermod [-;cdefgGasuLU]
    -l 修改账户名称
    -L lock
    -U unlock
    修改/etc/shadow
    -f 第七字段
    -e 第六字段
    -c /etc/passwd 第五字段
    -d /etc/passwd 主文件夹第六字段
    -g /etc/passwd 第四个字段GID
    -G 后面接次要用户组，修改这个用户能支持的用户组，修改/etc/group
    -a 与-G连用，增加次要用户组的支持而非设置
    -u UID /etc/passwd的第三个字段,UID
    -s 接shell的实际文件

#### 示例
~#:usermod -l 'my_usename' username


### user删除
#### 参数介绍
userdel 

### 示例
~#:userdel -r username # 删除主文件夹
~#:find / -user username
~#:userdel username

## finger查看用户的数据
finger 查看当前用户的数据
finger username 查看某用户的信息


## chfn
chfn 就是相当于-c参数,修改当前用户/etc/passwd的第五个字段值
chsh -s　修改当前用户的shell
chsh -s /bin/bash

## id
id [username]
列出当前用户或者username的所有id

## group操作
groupadd [-gr]
    -g　指定GID
    -r　新建系统用户组
groupmod [-gn]
    -n　修改组名
groupdel [groupname]

尽量少修改GID否则会造成系统资源的混乱
当用户组为某个用户的初始用户组时，就无法删除该用户组

### gpasswd修改group信息
gpasswd　[-AMrR] groupname
    -A 将groupname的控制权交给后面用户
        gpasswd -A mxx groupname
    -M 将某些账号加入到这个用户组中
    -r 将groupname的密码删除
    -R 将groupname的密码失效

gpasswd groupname 设置groupname管理密码
gpasswd groupname 
-A 增加groupname的管理员
-r让密码删除

gpasswd -ad username groupname
    -a增加
    -d删除


## ACL  Acess Control List
针对单一用户或者目录来进行rwx的权限设置
setfacl　[-m|-x]    -m设置acl参数   -x删除后续acl参数
    [-bkRd]
    -b删除所有的acl参数;-k删除默认的acl参数;-R递归设置acl;-d设置默认的acl，只对目录有效
setfacl [-m|-x] [bkRd]
    -b　删除所有ACL参数
    -k　删除默认ACL参数
    -R　递归设置ACL参数，包括子目录
    -d  设置默认ACL参数，只对目录有效
    
    -x　删除后续的ACL参数
    -m  设置后续的ACL参数

    -m u:mxx:rw my_file
        
getfacl my_file

针对有效权限mask的设置
setfacl -m m:rwx my_file
mask在此可以来规定最大允许的权限。取得是mask和用户以及用户组的权限交集。        若用户mxx的权限为rwx 但是mask为r--，那么mxx的权限只能为r--.

~#:setfacl -m d:u:mxx:rwx file　递归设置目录的acl
~#:setfacl -m m:rw acl_test
~#:setfacl -m g:mxx:rwx acl_test
~#:getfacl acl_test
~#:setfacl -b file 删除acl

## 切换用户，切换账号
su[- -l -m -c]
su - 切换到root用户以login shell变量的读取方式
su 切换到root用户，以nologin shell变量的读取方式登陆系统
｀｀    su -l 加想要切换的账号login shell   
su -c 只提升一次到root权限
su -m 使用目前用户的环境变量，不读取新用户的配置文件

su - -c cat /etc/shadow

sudo -u mxx ...提升到mxx权限

## visudo的设置
1.visudo 修改/etc/sudoers
其他用户使用root身份
root ALL=(ALL) ALL
用户账号
登陆者的来源主机名，
可切换的身份
可执行的命令
2.最左边加一个%表示用户组
利用用户组以及免密码
%wheel ALL=(ALL) ALL
usermod -a -G wheel user

免密
%wheel ALL=(ALL) NOPASSWD: ALL
3.mxx ALL=(root) /usr/bin/passwd
mxx可以切换到root的身份使用passwd命令

mxx ALL=(root) !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]\*,\
        !/usr/bin/passwd root
4.别名设置
User_Alias MYUSER=mxx,mahuihui
Cmnd_Alias MYCOMMAND=!/usr/bin/passwd, /usr/bin/passwd [A-Za-z]\*,\
        !/usr/bin/passwd root
MYUSER all=(root) MYCOMMAND

5.用自己的密码切换成root

sudo su -


## 用户信息传递
查询用户
- w   
- who 
- last 
- lastlog

用户对谈
mesg y
mesg n

write   
write mxx tty1

wall "hello"　每个人都会收到

mail mahuihui -s "Hi,mahuihui,nihaoa"
...
...
ctrl+d

mail 收信
?查看命令

q离开,离开后，会将该信件移动到~/home/mbox，收信箱
读取
mail -f /home/mxx/mbox  

## 手工添加账号
- pwck
pwconv将/etc/passwd相关信息移动到/etc/shadow,把在/etc/passwd中存在的账号            但是在/etc/shadow没有对应密码的列新增密码

- grpconv
- pwunconv    
- chpasswd    修改密码
echo "mxx:mypaswd" | chpasswd -m

### 新建账号
vim /etc/group
mygroup:x:1020:

grpconv

vim /etc/passwd
myuser:x:1200:1020::/home/myuser:/bin/bash
pwconv

passwd myuser

cp -a /etc/skel /home/myuser
chmod -R myuser:mygroup /home/myuser
chmod 700 /home/myuser  

## 参考文献
1.《鸟哥的LINUX私房菜》 

