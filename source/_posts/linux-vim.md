---
title: linux vim
date: 2019-05-07 16:58:22
tags: 
 - linux
 - vim
categories: linux
mathjax: true
---

## vim 配置文件
个人vim配置文件一般在~/.vimrc下，可以自定义各种配置。
[我的vimrc文件]()
### 前缀符号
为了缓解快捷键冲突问题，就引入了前缀键，跟参考文献[0]一样，设置;号为前缀键。
let mapleader=";"


### 设置显示行号
set number

### 底部显示文件路径
set laststatus=2 "设置底部状态栏可见
set statusline=%F%m%r%h%w\ %=#%n\ [%{&fileformat}:%{(&fenc==\"\"?&enc:&fenc)    .((exists(\"\+bomb\")\ &&\ &bomb)?\"\+B\":\"\").\"\"}:%{strlen(&ft)?&ft:'\*\*'    }]\ [%L\\%l,%c]\ %p%%    "statusline显示的信息，来自参考文献[8]。
"其中%L是当前文件缓冲区的行数，%P是当前行占总行数的百分比。
set ruler "显示光标当前位置

## vim复制到系统寄存器
### vim寄存器
vim有9种寄存器:
1. "是未命名寄存器，vim的默认寄存器，存放删除和复制的文本。
2. small delete寄存器 -，存放不超过一行的delete操作（不包括x操作）产生的文本。
3. 编号为$0,1,2,\cdot, 9$的寄存器，
4. $a-za-z$的$26$个字母寄存器,
5. 只读寄存器 : .,%,\$
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

## vim模式和常用命令
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

#### 查找
/word ?word
n N

#### 替换和删除
:1,10s/word/word.rp/g(c) 
:1,$s/word/word.rp/g(c)
利用正则表达式可以实现下面的一些常用命令
[代码地址](https://github.com/mxxhcm/code/tree/master/shell/vim_regex)
``` shell
# 替换所有的^为\^
:0,$s/\^/\\^/g
# 替换所有的\*\*为##
:0,$s/[0-9][0-9]\./## /g
# 删除所有以tab开头的tab
:0,$s/^\t//g
# 删除所有以#开头的行
:g/^#\.\*$/d
# 删除所有空行
:g/^\s\*$/d
# 用newlines替换,
:0,$s/,/\r/g
```

#### 其他
:w [filename]
:r [filename]
:n1,n2 w [filename]
:set nu

### Visual模式
见参考文献[9]。

## 快捷键映射
- namp 正常模式下的递归映射
- vmap Visual模式
- imap 插入模式
- cmap 命令模式
- nnoremap 正常模式下的非递归映射
- vnoremap Visual模式下的非递归映射
- inoremap 插入模式下的非递归映射
- cnoremap 命令模式下的非递归映射

## 其他vim使用事项
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

## 问题-vim中设置了setexpand不起作用
~/.vimrc中进行了如下设置：
``` vimrc
set expandtab
set tabstop=4
set shiftwidth=4
set softtabstop=4           
```
但是发现在markdown甚至~/.vimrc中expandtab都没有设置成功，但是py文件是正常的，后来发现是多加了一个set paste的原因，把它删了就好了。

### 原因
因为set paste覆盖了set expandtab。

### 解决方案
删除set paste行。

## 我的vimrc文件
vimrc文件如下，[代码地址](https://github.com/mxxhcm/code/blob/master/shell/vimrc)
``` shell
# 使用四个空格代替tab键
set expandtab
set tabstop=4
set shiftwidth=4
set softtabstop=4	
			
" 打开文件类型检测
filetype on
" 根据不同的文件类型加载插件
filetype plugin on
set ignorecase

" 定义前缀键
" let mappleader=";"

" 设置ctrl+a全选，ctrl+c复制，;y粘贴
vnoremap ; "+
nnoremap ; "+
nmap ;p "+p
nnoremap <C-C> "+y
vnoremap <C-C> "+y
nnoremap <C-A> ggVG
vnoremap <C-A> ggVG

" 删除#号开头
nnoremap ;d3 :g/^#.*$/d<CR>
nnoremap ;d# :g/^#.*$/d<CR>
" 删除空行
nnoremap ;ds :g/^\s*$/d<CR>
" 删除以tab开头的tab
nnoremap ;rt :0,$s/^\t//g<CR>
" 用\^代替^
nnoremap ;r6 :0,$s/\^/\\^/g<CR>
nnoremap ;r^ :0,$s/\^/\\^/g<CR>
" 用\\\\代替\\
nnoremap ;r/ :0,$s/\\\\/\\\\\\\\/g<CR>
nnoremap ;r? :0,$s/\\\\/\\\\\\\\/g<CR>

" 给选中行加注释
" cnoremap <C-#> s/^/# /g<CR>
nmap ;ic :s/^/# /g<CR>
vmap ;ic :s/^/# /g<CR>
nmap ;dc :s/^# //g<CR>
vmap ;dc :s/^# //g<CR>
"vmap <C-#> :s/^/#/g<CR>
"nmap <C-#> :s/^/#/g<CR>

""" 状态栏设置
" 总是显示状态栏
set laststatus=2
" 状态信息
set statusline=%f%m%r%h%w\ %=#%n\ [%{&fileformat}:%{(&fenc==\"\"?&enc:&fenc).((exists(\"\+bomb\")\ &&\ &bomb)?\"\+B\":\"\").\"\"}:%{strlen(&ft)?&ft:'**'}]\ [%c,%l/%L]\ %p%%

"""光标设置
" 设置显示光标当前位置
set ruler

" 开启行号显示
set number
" 高亮显示当前行/列
set cursorline
" set cursorcolumn
" 高亮显示搜索结果
set hlsearch
" 显示文件名

" 开启语法高亮
syntax enable
" 允许用指定语法高亮配色方案替换默认方案
syntax on
" 将制表符扩展为空格
" 设置编辑时制表符占用空格数
" 设置格式化时制表符占用空格数
" 让 vim 把连续数量的空格视为一个制表符
set autoindent
```

## 参考文献
0.https://github.com/yangyangwithgnu/use_vim_as_ide
1.https://askubuntu.com/a/1027647
2.https://www.zhihu.com/question/19863631/answer/89354508 
3.鸟哥的LINUX私房菜
4.http://vimdoc.sourceforge.net/htmldoc/change.html#registers
5.https://stackoverflow.com/a/11489440
6.https://www.brianstorti.com/vim-registers/
7.http://landcareweb.com/questions/3593/ru-he-zai-vimzhong-yong-jiu-xian-shi-dang-qian-wen-jian-de-lu-jing
8.https://forum.ubuntu.org.cn/viewtopic.php?t=319408
9.https://stackoverflow.com/a/1676659/8939281<op selected lines>
10.https://vi.stackexchange.com/questions/9028/what-is-the-command-for-select-all-in-vim-and-vsvim/9029<ctrl A>
11.https://stackoverflow.com/a/37962622/8939281<set paste>
12.https://vim.fandom.com/wiki/Search_and_replace_in_a_visual_selection
13.https://stackoverflow.com/questions/71323/how-to-replace-a-character-by-a-newline-in-vim/71334<用newline替换逗号>
