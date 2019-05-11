---
title: linux vim
date: 2019-05-07 16:58:22
tags: 
 - linux
categories: linux
mathjax: true
---

## vim复制到系统寄存器
### vim寄存器
vim有9种寄存器:
1. "是未命名寄存器，vim的默认寄存器，存放删除和复制的文本。
2. small delete寄存器 -，存放不超过一行的delete操作（不包括x操作）产生的文本。
3. 编号为$0,1,2,\cdot, 9$的寄存器，
4. $a-za-z$的$26$个字母寄存器,
5. 只读寄存器 : . % $
6. 表达式寄存器 =
7. 搜索寄存器 /
8. GUI选择寄存器$\*,+$。
9. 黑洞寄存器，向这个寄存器写入的话，什么都不会发生。

详细介绍可见参考文献[4]。

### 使用系统剪切板
\*和+寄存器适合系统相关的，\*和系统缓冲区关联，+和系统剪切板关联。
使用+y复制当前行到系统剪切板。
使用+ny复制n行到系统剪切板。
使用+p粘贴系统剪切板到当前位置。 
但是有些vim发行版不支持系统剪切板，可以使用如下命令查看自己的系统是否支持系统剪切板。
~\\$:vim --version|grep clipboard
在我的系统上，输出如下：
> -clipboard         +jumplist          +persistent_undo   +virtualedit
-ebcdic            -mouseshape        +statusline        -xterm_clipboard

说明当前vim不支持系统剪切板，所以就卸载安装支持的版本呗。
~\\$:sudo apt remove vim
~\\$:sudo apt install vim-gtk3
然后再次查看
~\\$:vim --version|grep clipboard
输出如下：
> +clipboard         +jumplist          +persistent_undo   +virtualedit
-ebcdic            +mouseshape        +statusline        +xterm_clipboard

说明已经支持系统剪切板，可以使用了。注意记得把之前打开的vim关闭后再试。
使用以下命令进行操作：
+nyy # 复制从当前行开始的n行到+寄存器
+yy # 复制当前行行到+寄存器
+p # 粘贴+寄存器中的内容到文本中。
这个时候还有一个问题，就是一般的笔记本键盘的+和=号是在一起的，如果要打出=行，需要按一下shit +=，这个时候会向下移动一行，但是无伤大雅，为什么会这样，我还不知道。详细流程可参见参考文献[5]。
但是后来我发现这个还不能用。然后就只能继续查找了。在知乎上找到一个回答，发现还要在这些命令前加上一个"号，表示将默认"寄存器中的内容复制到+寄存器中。也就是使用如下命令：
"+nyy # 复制从当前行开始的n行到+寄存器
"+yy # 复制当前行行到+寄存器
"+p # 粘贴+寄存器中的内容到文本中。

### 将未命名寄存器和系统寄存器设为同一个。
修改vim配置文件
~$:vim ~/.vimrc
添加下面一句话，重新打开vim即可
set clipboard=unnamed

## 常用的vim命令
### vim模式
- 正常模式，用vim打开一个文件之后就处于正常模式
- 插入模式，在正常模式下输入i,a,o或者I,A,O之后，就进入了命令模式，可以修改文件，按Esc退出。
- Visual模式，可以移动光标选中某些行，进行复制或者删除，在正常模式按v或者V进入Visual模式。
- 命令模式，在正常模式按:进入命令模式，可以在窗口底部输入命令。
- 替换模式，使用r替换当前字符，使用R从当前字符开始连续替换。

### 正常模式

#### 移动光标
0 移动到行首
$ 移动到行尾
h 向左移动一个character 
j 向下移动一行
k 向上移动一行
l 向右移动一个character
nj nk nh nl 移动n次
oO o在当前行的下一行插入，O在当前行的上一行插入
iI i在当前光标处插入，I在行首插入
aA a在当前光标后插入，A在行尾插入
1G 跳到第一行
gg 跳到首行
G 跳到尾行
nG 跳到尾行
n-space 跳到光标后第n个character
n-ENTER nG 跳到第n行
wW w移动到下一个word的开头，W移动到隔了一个空白符的下一个word的开头
bB b移动到前一个word的开头，B移动到隔了一个空白符的前一个word开头
eE 移动到当前word的结尾，W移动到隔了一个空白符的word结尾。
ctrl+f 跳到下一页 
ctrl+b 跳到上一页

#### 删除和复制
x 删除一个character
nx 删除n个characters
dd 删除当前行
ndd 删除n行
yy 复制一行
nyy 复制n行
p 粘贴

### 命令模式
#### 切屏
:sp [filename]
输入:进入命令模式，然后输入sp，空格，要打开的文件名。使用ctrl w在分开的屏幕之间进行切换。

#### 替换
:1,10s/word/word.rp/g(c)
:1,$s/word/word.rp/g(c)

#### 查找
/word ?word
n N

#### 其他
:w [filename]
:r [filename]
:n1,n2 w [filename]
:set nu

## 其他vim使用事项(290)
### 编码
tty1-tty6默认不支持中文编码　　
修改终端接口语系　
LANG=zh_CN.big5

### dos和UNIX转换
ubuntu  don't have dos2UNIX or UNIX2dos  but is has tofrodos

frodos filename
todos filename
    -b .bak
    -v ver

### 语系编码转换
iconv -o保留原文件，-o加新文件名
iconv -f big5 -t utf8 filename -o filename

## 参考文献
1.https://askubuntu.com/a/1027647
2.https://www.zhihu.com/question/19863631/answer/89354508 
3.鸟哥的LINUX私房菜
4.http://vimdoc.sourceforge.net/htmldoc/change.html#registers
5.https://stackoverflow.com/a/11489440
6.https://www.brianstorti.com/vim-registers/
