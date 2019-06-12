---
title: linux at cron anacron mail
date: 2019-05-07 16:50:33
tags: 
 - linux
categories: linux
---


## 例行性工作
- at    仅执行一次
- cron  周期性执行
- anacron   适合不常开机的设置

## at仅执行一次的工作调度
### 参数说明
at [-lmdvc] TIME
    -m 当at完成时，即使没有输出信息，以mail通知用户
    -v 可以使用较明显的时间格式列出at调度中的工作
    -c 可以列出后面接的该项工作的实际命令内容
    -d 相当于atrm，可以取消一个at工作
    -l 相当于atq，列出目前系统上所有该用户的at调度
    -b 相当于batch
    TIME
        HH:MM	04:00
        HH:MM YYYY-MM-DD    05:00 2016-10-05
        HH:MM[pm|am] [Month] [Date] 04 January 10
        HH:MM [am|pm] + number [minutes|hours|days|weeks]   
            now + 5 minutes
            05pm + 3days
            04pm + 10 days

### 示例
#### 创建一个job
~$:at now+1minutes
at\>echo "create a job"
按ctrl+D结束
OK，但是这样子我找不到任何程序的输出在哪里。
所以可以改成这样子
at\>echo "create a job" \> at_job.output
或者
~$:echo "create a job" \> at_job.output | at now

#### 列出所有at jobs
~$:at -l   # 列出at的所有任务
~$:atq

#### 列出某个job
~$:at -c [number](1, 2..) # 如果当前没有相应的job，会输出cannot find jobid x

#### 删除某个job
~$:at -r 8
~$:atrm 1

### 配置文件
/etc/at.allow   # 哪些人能使用
/etc/at.deny    # 哪些人不能使用
使用at命令的话，先查找at.allow，如果存在并且有内容，那么只有这些人能使用。如果不存在的话，就去找at.deny。

## batch 
当空闲时执行，空闲指的是CPU占用率在$0.8$以下

## crond例行性工作调度
### 参数介绍
crontab [-u user] [-ler]
    -u　只有root能设置这个参数
    -l　列出当前用户的所有crontab工作内容
    -e  编辑crontab的内容
    -r　删除所有crontab的内容

### 示例
#### 新建crontab
**注意：周与月日不可共存**
~$:crontab -e
\* \* \* \* \* cmd
分钟　小时　日期　月份　星期
\*表示任何取值，
\-表示时间范围 0-59,
","表示分隔 3,6,9
"/n"，如\*/5每过五个单位(分钟，小时，天)
比如添加每一小时给荟荟发一封邮件，需要添加以下内容
``` txt
* */1 * * * echo "I love you." | mail -s "huhui" 18811367922@163.com
```

#### 删除一个crontab
~$:crontab -e
然后手动编辑要删除的crontab

#### 删除所有crontab
~$:crontab -r # 删除所有的crontab

### 开启/var/log/cron.log
~$:vim /etc/rsyslog.d/50-default.conf
将rsylog文件中的#cron.\*前的#去掉
~$:service rsyslog restart
~$:service cron restart
~$:vim /var/log/cron.log

### 系统任务
/etc/crontab 为系统的例行性任务，它会执行以下run-parts
- /etc/cron.daily/
- /etc/cron.hourly/
- /etc/cron.monthly/
- /etc/cron.weekly/

### 自定义run-parts
直接编辑/etc/crontab文件，在其中添加
``` txt
# m h dom mon dow user	command
\*/2 \* \* \* \* root   run-parts /etc/cron.minutely
\*/5 \* \* \* \* root   run-parts /root/runcron
# 上述两条命令中，需要对应的目录存在或者直接执行一个shell脚本
\* \* \* \* \* mxxmhh /bin/bash /home/mxxmhh/outputtime_minutes.sh
```
outputtime_minutes.sh脚本如下
``` /home/mxxmhh/outputtime_minutes.sh
#! /bin/bash
time=`date`
echo $time >> /home/mxxmhh/test.log
```

### crontab -e vs vim /etc/crontab
他们的格式不同，一个需要指定用户，一个不需要
只有root能够修改/etc/crontab，而crontab -e所有不在cron.deny中的用户都可以
/etc/crontab是系统的任务，crontab -e是用户的任务

### 配置文件
ubuntu中没有下面两个配置项
/etc/cron.allow
/etc/cron.deny
即默认为所有用户都可以使用crontab

### cron spool
/var/spool/cron/crontabs/
该目录下为不同账号的crontab内容

## anacron 处理非24小时开机的系统
### 参数介绍
anacron [-usfn] [job]
    -u 更新记录文件的时间戳
    -s 开始连续执行各项job，依据记录文件的时间戳判断是否进行
    -f 强制执行，不管时间戳
    -n 立即进行未进行的任务，而不延迟

### 示例
系统的anacron文件都在目录/etc/cron\*/\*ana\*存放
/etc/cron.daily/0anacron
0表示最先被执行，让时间戳先被更新，避免anacron误判
/etc/anacron	anacron的设置

/var/spool/anacron/\*	
记录最近一次执行anacron的时间戳

## mail命令介绍
mail -s "title" target_email_address
echo "content |mail -s "title" target_email_address
mail -s "title target_email_address < file #将file的内容当做邮件正文

## mail发送邮件
### 安装相应软件
~$:sudo apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules
编辑/etc/postfix/main.cf文件，在文件末尾添加下列内容
``` txt
# 指定默认的邮件发送服务器
relayhost = [smtp.gmail.com]:587
# 激活sasl认证
smtp_sasl_auth_enable = yes
# 指定sasl密码配置文件
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# 非匿名登录
smtp_sasl_security_options = noanonymous
# linux用户与发件人的对应关系配置文件
sender_canonical_maps = hash:/etc/postfix/sender_canonical 
smtp_tls_CApath = /etc/ssl/certs
smtpd_tls_CApath = /etc/ssl/certs
smtp_use_tls = yes
```

### 创建密码配置文件
~$:vim /etc/postfix/sasl_passwd
添加如下内容
``` txt
# 163邮箱格式
[smtp.163.com]:25 your163mail:your163mailpassword   #注意这里如果直接用passwd是会报错的，需要使用授权码
# gamil邮箱格式
[smtp.gmail.com]:587 yourgmail:yourgmailpassword
```
~$:sudo postmap /etc/postfix/sasl_passwd

### 创建用户与发件人对应文件
~$:vim /etc/postfix/sender_canonical
添加如下内容
``` txt
root your163mail
user1 yourgmail
```
~$:sudo postmap /etc/postfix/sender_canonical

### 重启postfix服务
~$:sudo /etc/init.d/postfix reload
或者
~$:sudo systemctl relaod postfix.service
或者
~$:sudo service postfix restart

### 测试
~$:echo "Hello." |mail -s "I love you." 18811376816@163.com

## 附录
更多at命令的TIME格式
``` txt
noon	
midnight	
teatime	
tomorrow	
noon tomorrow	
next week	
next monday	
fri	
NOV	
9:00 AM	
2:30 PM	
1430	
2:30 PM tomorrow	
2:30 PM next month	
2:30 PM Fri	
2:30 PM 10/21	
2:30 PM Oct 21	
2:30 PM 10/21/2014	
2:30 PM 21.10.14	
now + 30 minutes	
now + 1 hour	
now + 2 days	
4 PM + 2 days	
now + 3 weeks	
now + 4 months	
now + 5 years	
```

## 参考文献
1.《鸟哥的LINUX私房菜》
2.https://zhidao.baidu.com/question/249718018.html
3.https://askubuntu.com/questions/1112772/send-system-mail-ubuntu-18-04
4.https://www.cnblogs.com/tugeler/p/6620150.html
