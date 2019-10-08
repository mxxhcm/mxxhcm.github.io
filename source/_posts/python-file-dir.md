---
title: python文件和目录操作(os和shutil)
date: 2019-04-13 14:51:26
tags:
 - python
categories: python
---

## 文件和目录操作（os库和shutil库）

import os
### 查看信息
不是函数，而是属性
os.linesep   #列出当前平台的行终止符
os.name    #列出当前的平台信息

### 列出目录 
file_dir_list = os.listdir(parent_dir)    #列出某个目录下的文件和目录，默认的话为当前目录
parent_dir 是一个目录
file_dir_list是一个list

os.path.exists(pathname)    #判断pathname是否存在
os.path.isdir(pathname)    #判断pathname是否是目录
os.path.isfile(pathname)    #判断pathname是否是文件
os.path.isabs(pathname)    #判断pathname是否是绝对路径

os.path.basename(pathname)    # 列出pathname的dir
os.path.dirname(pathname)        # 列出pathname的file name
os.path.split(pathname)    #将pathname分为dir和filename
os.path.split(pathname)    #将pathname的扩展名分离出来

os.path.join("dir_name","file_name")    # 拼接两个路径

os.getcwd()    #获得当前路径
os.chdir(pathname)    #改变当前路径
os.path.expanduser(pathname)    #如果pathname中包含"~"，将其替换成/homre/user/


### 创建和删除 
os.mkdir(pathname)    #创建新目录
os.rmdir(pathname)    #删除目录
os.makedirs("/home/mxxhcm/Documents/")    #创建多级目录
os.removedirs()    #删除多个目录
os.remove(file_pathname)    #删除文件

os.rename(old_pathname,new_pathname)    #重命名

### 打开文件
对于open文件来说，共有三种模式，分别为w,a,r
r的话，为只读，读取一个不存在的文件，会报错
r+的话，为可读写，读取一个不存在的文件，会报错
a的话，为追加读，读取一个不存在的文件，会创建该文件
w的话，为写入文件，读取一个不存在的文件，会创建改文件，打开一个存在的同名文件，会删除该文件，创建一个新的文件


### 读取文件
fp = open(file_path_name,"r+")
#### read()将文件读到一个字符串中
file_str = fp.read()
fp.read()会返回一个字符串，包含换行符

#### readline()
for file_str in fp:
    print(file_str)
这里的file_str是一个str类型变量

#### readlines()将文件读到一个列表中
list(fp)
file_list = fp.readlines()
filt_list是一个list变量

### 关闭文件
fp.close()
或者
with open(file_pathname, "r") as f:
    file_str = fp.read()
当跳出这个语句块的时候，文件已经别关闭了。

### 复制文件
shutil.move('test','test_move')    # 递归的将文件或者目录移动到另一个位置。如果目标位置是一个目录，移动到这个目录里，如果目标已经存在而且不是一个目录，可能会用os.rename()重命名
shutil.copyfile(src,dst) #复制文件内容，metadata没有复制
shutil.copymode(src,dst) #copy权限。文件内容，owner和group不变。
shutil.copystat(src,dst)    #copy权限，各种时间以及flags位。文件内容，owner，group不变
shutil.copy(src,dst)    #copy file,权限为也会被copied
shutil.copy2(src,dst)  #和先后调用shutil.copy()和shutil.copystat()函数一样
shutil.copytree(src,dst,symlinks=False,ignore=None)  #递归的将str目录结构复制到dst，dst位置必须不存在，目录的权限和时间用copystat来复制，文件的赋值用copy2()来复制
shutil.rmtree(path[,ignore_errors[,onerror]])   #删除一个完整的目录，无论目录是否为空

## 参考文献
1.https://www.zhihu.com/question/48161511/answer/445852429


