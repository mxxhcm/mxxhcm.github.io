---
title: jupyter notebook笔记
date: 2019-03-18 15:14:33
tags:
 - python
 - jupyter
categories: 工具
---
## 一、安装和运行
### 1.安装
#### Anaconda安装
Anaconda自身已经集成了jupyter包，所以如果没有装python的话，可以选择安装Anaconda集成环境
#### pip安装
~#:pip install jupyter
### 2.运行
~#:jupyter notebook
### 3.远程访问
#### (1).直接使用命令
这种方法是建立了一个session，会有一个token，这个会话结束之后，这个token就无效了，需要再重现建立新的session
##### a.在前台运行以下命令
~#:jupyter notebook --ip=your_server_ip
输出如下

复制这个url到你的客户端浏览器，就可以直接访问服务器端。
##### b.后台运行
~#:nohup jupyter notebook --ip=10.4.21.214 &

#### (2).创建配置文件
##### a.服务器端设置密码
这里是使用notebook的passwd()函数生成自己设置密码的sha1哈希值。
``` python
from notebook.auth import passwd
passwd()
```
输入两边自己设置的密码，然后将哈希值复制到下面的配置文件中即可。

##### b.服务端设置配置文件
~#:jupyter notebook --generate-config
~#:vim ~/.jupyter/jupyter_notebook_config.py
``` text
c.NotebookApp.ip='localhost'
c.NotebookApp.password=u'sha1:...'
c.NotebookApp.open_browser=False
c.NotebookApp.port=8888(your_port)
```

##### c.服务器端启动
~#:jupyter notebook

##### d.客户端访问
http://your_server_ip:port
输入密码即可

## 二、使用
### 1.创建新的文档


## 三、快捷键
Jupyter Notebook 有两种键盘输入模式。编辑模式，允许你往单元中键入代码或文本；这时的单元框线是绿色的。命令模式，键盘输入运行程序命令；这时的单元框线是灰色。

### 1.命令模式 (按键 Esc 开启)
Enter : 转入编辑模式
Shift-Enter : 运行本单元，选中下个单元
Ctrl-Enter : 运行本单元
Alt-Enter : 运行本单元，在其下插入新单元
Y : 单元转入代码状态
M :单元转入markdown状态
R : 单元转入raw状态
1 : 设定 1 级标题
2 : 设定 2 级标题
3 : 设定 3 级标题
4 : 设定 4 级标题
5 : 设定 5 级标题
6 : 设定 6 级标题
Up : 选中上方单元
K : 选中上方单元
Down : 选中下方单元
J : 选中下方单元
Shift-K : 扩大选中上方单元
Shift-J : 扩大选中下方单元
A : 在上方插入新单元
B : 在下方插入新单元
X : 剪切选中的单元
C : 复制选中的单元
Shift-V : 粘贴到上方单元
V : 粘贴到下方单元
Z : 恢复删除的最后一个单元
dd : 删除选中的单元
Shift-M : 合并选中的单元
Ctrl-S : 文件存盘
S : 文件存盘
L : 转换行号
O : 转换输出
Shift-O : 转换输出滚动
Esc : 关闭页面
Q : 关闭页面
H : 显示快捷键帮助
I,I : 中断Notebook内核
0,0 : 重启Notebook内核
Shift : 忽略
Shift-Space : 向上滚动
Space : 向下滚动

### 2.编辑模式 ( Enter 键启动)
Tab : 代码补全或缩进
Shift-Tab : 提示
Ctrl-] : 缩进
Ctrl-[ : 解除缩进
Ctrl-A : 全选
Ctrl-Z : 复原
Ctrl-Shift-Z : 再做
Ctrl-Y : 再做
Ctrl-Home : 跳到单元开头
Ctrl-Up : 跳到单元开头
Ctrl-End : 跳到单元末尾
Ctrl-Down : 跳到单元末尾
Ctrl-Left : 跳到左边一个字首
Ctrl-Right : 跳到右边一个字首
Ctrl-Backspace : 删除前面一个字
Ctrl-Delete : 删除后面一个字
Esc : 进入命令模式
Ctrl-M : 进入命令模式
Shift-Enter : 运行本单元，选中下一单元
Ctrl-Enter : 运行本单元
Alt-Enter : 运行本单元，在下面插入一单元
Ctrl-Shift-- : 分割单元
Ctrl-Shift-Subtract : 分割单元
Ctrl-S : 文件存盘
Shift : 忽略
Up : 光标上移或转入上一单元
Down :光标下移或转入下一单元

