---
title: linux stdin stdout和stderr
date: 2019-06-03 00:09:27
tags:
- linux
categories: linux
---

## stdin, stdout, stderr 
standard input(file handle 0)：标准输入，process利用这个file handle从用户读取数据
standard output(file handle 1)：标准输出，process向这个file handle写normal infor，会将输出输出到屏幕
standard error(file handle 2)：标准错误，process向这个file hanle写error infor

## 数据流重定向
>, >>, <, <<

### 输出重定向
>是将std output进行重定向，>>是将std output进行追加。如果要将std error重定向或者追加，需要使用2表示表示std error。
~$:ls -ld /etc/ >~/etc_dir.txt

#### std output 重定向
~$:find / -name ~/.bashrc >find_result 

#### std errort重定向
~$:find / -name ~/.bashrc >fing_right 2> find_error
~$:find / -name ~/.bashrc > find_result 2>/dev/null
~$:find / -name ~/.bashrc >find_result 2>&1


### 输入重定向
#### \<从键盘读入
~$:cat \> catfile
\>
\>
\> ctrl + D

#### 从文件中读入数据
cat > catfile < ~/.bashrc

#### 重定义输入结束符
cat > catfile << "eof"

## 示例
### 示例1
~$:my_command \<inputfile 2\>errorfile | grep XYZ
执行my_command，
打开inputfile文件作为标准输入，
打开errorfile文件作为标准错误，
|重定向上述命令的结果到grep 命令

### 示例2
``` shell
echo "hello"
# 是将"hello"给了echo的stdin的简写
echo < text.txt
# 将text.txt给echo的stdin
ps | grep 1000
# |将ps的stdout给了grep的stdin
ls . > current_dir_list.txt
# >将ls .的stdout输出到相应文件
ls. >> current_dir_list.txt
# >>是追加，>不是
```

## 2>&1和&>
```
command 2>&1 output_error.txt，
#可以将2>&1看成将stderr重定向到stdout，如果写成2>1的话看起来像是将stderr重定向到一个名为$1$的文件。在redirece的上下文中，&可以看成file descriptor的意思。为什么不写成&2>&1，这会被解析成& 和2&1，第一个&会被解析成后台运行，然后剩下的就是2>&1了。
# 将command的stdout和stderr都输出到该文件
command &> /dev/null
# 将command的stderr和stdout输出到/dev/null，将会什么也不输出
```
&>是bash的扩展，而2>&1是standard Bourne/POSIX shell。

## 一个神奇的命令\</dev/null &>
/dev/null是一个神奇的文件，它代表null device，会抛弃所有写入它的数据，但是会report写操作成功了，并且它不会向读取它的任何process提供任何数据，也就是向读取它的程序发送一个EOF。
所以\</dev/null会发送一个EOF到stdin，&就是将程序放在后台运行，&的详细介绍可见[linux process章节]()
这时候如果再加一个重定向命令，就是将命令的输出重定向到某个文件中而不在stdout中显示。

## 示例
~$:nohup sslocal -c /etc/shadowsocks_v6.json \</dev/null &\>\>~/.log/ss-local.log &  # 后台运行sslocal
~$:nohup /home/mxxmhh/anaconda3/bin/python main.py >log_name 2>&1 &

## 参考文献
1.https://unix.stackexchange.com/questions/27955/the-usage-of-dev-null-in-the-command-line
2.https://stackoverflow.com/questions/3385201/confused-about-stdin-stdout-and-stderr
3.https://stackoverflow.com/questions/818255/in-the-shell-what-does-21-mean
4.https://askubuntu.com/questions/635065/what-is-the-differences-between-and-21
