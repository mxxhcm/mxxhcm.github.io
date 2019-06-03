---
title: linux custom configure file
date: 2019-06-03 10:40:12
tags:
- linux
categories: linux
---

## bashrc自定义配置
~/.bashrc文件详细内容，[代码地址](https://github.com/mxxhcm/code/blob/master/shell/bashrc)
``` shell
# dir 
alias post-dir='/home/mxxmhh/mxxhcm/blog/source/_posts'
alias code-dir='/home/mxxmhh/mxxhcm/code/'
alias matplotlib-dir='/home/mxxmhh/mxxhcm/code/tools/matplotlib/'
alias numpy-dir='/home/mxxmhh/mxxhcm/code/tools/numpy/'
alias shell-dir='/home/mxxmhh/mxxhcm/code/shell'
alias github-dir='/home/mxxmhh/github/'
alias torch-dir='/home/mxxmhh/mxxhcm/code/pytorch'
alias tf-dir='/home/mxxmhh/mxxhcm/code/tf'
alias rl-dir='/home/mxxmhh/mxxhcm/code/rl'
alias ops-dir='/home/mxxmhh/mxxhcm/code/tf/ops'
alias paper-dir='/home/mxxmhh/mxxhcm/papers'
alias tf-dir='/home/mxxmhh/mxxhcm/code/tf'

# cd dir
alias post='cd /home/mxxmhh/mxxhcm/blog/source/_posts'
alias code='cd /home/mxxmhh/mxxhcm/code/'
alias matplotlib='cd /home/mxxmhh/mxxhcm/code/tools/matplotlib/'
alias numpy='cd /home/mxxmhh/mxxhcm/code/tools/numpy/'
alias shell='cd /home/mxxmhh/mxxhcm/code/shell'
alias github='cd /home/mxxmhh/github/'
alias torch='cd /home/mxxmhh/mxxhcm/code/pytorch'
alias tf='cd /home/mxxmhh/mxxhcm/code/tf'
alias rl='cd /home/mxxmhh/mxxhcm/code/rl'
alias ops='cd /home/mxxmhh/mxxhcm/code/tf/ops'
alias paper='cd /home/mxxmhh/mxxhcm/papers'
alias update='hexo g -d'
alias n='hexo n '
alias new='hexo n '
alias status='git status'
alias add='git add .'
alias remove='git rm'
alias commit='git commit -m '
alias branch='git branch'
alias check='git checkout '
alias push-master='git push origin master'
alias pull-master='git pull origin master'
alias push-hexo='git push origin hexo'
alias pull-hexo='git pull origin hexo'
alias utorrent='utserver -settingspath /opt/utorrent/'
alias ssr='nohup sslocal -c /etc/shadowsocks_v6.json </dev/null &>>~/.log/ss-local.log & '
alias ssr4='nohup sslocal -c /etc/shadowsocks_v4.json </dev/null &>>~/.log/ss-local.log & '
alias ssr5='nohup sslocal -c /etc/shadowsocks_v6.json </dev/null &>>~/.log/ss-local.log & '
alias ssr6='nohup sslocal -c /etc/shadowsocks_v6.json </dev/null &>>~/.log/ss-local.log & '

function sspid(){
	ps aux |grep 'sslocal'
}

function killss(){
	pid=`ps -A | grep 'sslocal' |awk '{print $1}'`
	echo $pid
	kill -9 $pid
}

function anaconda_on(){
	export PATH=/home/mxxmhh/anaconda3/bin:$PATH
}

function vultr(){
	ssh root@2001:19f0:7001:20f8:5400:01ff:fee6:aff6
}

function deploy-upload-hexo(){
	# echo "pull"
	# git pull origin hexo
	echo "push"
	git add .
	git commit -m "update blog"
	git push origin hexo
	hexo g -d
}

function upload-master(){
	# echo "pull"
	# git pull origin master
	echo "push"
	git add .
	git commit -m "update code"
	git push origin master
}

function folder-size(){
	dir=`pwd`
	du -h --max-depth=1
}

export PATH=/home/mxxmhh/anaconda3/bin:$PATH
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64/:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda


export https_proxy="http://127.0.0.1:8118"
export http_proxy="http://127.0.0.1:8118"
export HTTP_PROXY="http://127.0.0.1:8118"
export HTTPS_PROXY="http://127.0.0.1:8118"

function proxyv6_off(){
	unset https_proxy
	unset http_proxy
	unset HTTP_PROXY
	unset HTTPS_PROXY
}
```

## vim自定义配置
关于vim详细介绍，可以查看[linux-vim](https://mxxhcm.github.io/2019/05/07/linux-vim/)
vimrc文件如下，[代码地址](https://github.com/mxxhcm/code/blob/master/shell/vimrc)
``` vimrc
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

" 设置复制和粘贴到系统粘贴板的快捷键
vnoremap ; "+
nnoremap ; "+
nmap ;p "+p
nnoremap <C-C> "+y
vnoremap <C-C> "+y

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
"
nnoremap ;r/ :0,$s/\\\\/\\\\\\\\/g<CR>
nnoremap ;r? :0,$s/\\\\/\\\\\\\\/g<CR>

" 给选中行加注释
" cnoremap <C-#> s/^/# /g<CR>
nmap ;ic :s/^/# /g<CR>
vmap ;ic :s/^/# /g<CR>
nmap ;rc :s/^# //g<CR>
vmap ;rc :s/^# //g<CR>
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
set paste	
```

## 参考文献
1.《鸟哥的LINUX私房菜》基础篇
2.https://github.com/yangyangwithgnu/use_vim_as_ide