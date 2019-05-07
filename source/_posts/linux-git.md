---
title: linux git
date: 2019-05-07 16:56:44
tags:
 - linux
categories: linux
---


27.how to use github in ubuntu
	
	安装github
	sudo apt-get install git git-core git-gui
	检查ssh
	ssh -T git@gitgub.com
	如果看到
	Warning: Permanently added ‘github.com,204.232.175.90’ (RSA) to the list 		of known hosts. 
	Permission denied (publickey).
	则可以继续
	~$:cd ~/.ssh
	查看是否有ssh keys
	并没有发现执行下一步	
	~$:ssh-keygen -t rsa -C "github 邮箱"
	在github上登陆自己的账号，找到Settings->SSH Keys -> ADD SSH Key 将
		is_rsa.pub文件中的字符串复制进去，不含空格和回车。
	~$:ssh -T git$github.com

	git config  --gloabl user.name "github用户名"
	git config  --gloabl user.email "github邮箱"

	在github上新建repository 名为 testgit
	~$:cd mkdir /home/mxx/github/testgit
	~$:cd github/testgit
	~$:git init
	~$:touch Readme
	~$:git add Readme
	~$:git commit -m 'add readme file'
		版本回退
		~$:git reset --hard HEAD^ 回退到上个版本 
		~$:git reflog 查看提交的id
		~$:git reset --hard id
		管理修改
		 在工作区修改后回退　
		~$:vim file
		~$:git checkout -- file
		 加入缓存区后回退
		~$:git add file
		~$:git status
		~$:git reset HEAD file 清除了缓存去但是工作区还有修改
		~$:git status
		~$:git checkout -- file 清除工作区修改
		 加入版本库后回退
		~$:git commit -m "add file"
		~$:git reset --hard HEAD^ 但是在提交到远程库后不能撤销。。
		删除文件
		~$:rm test
		~$:git status 查看状态
		~$:git rm test
		~$:git commit -m "delete file test"
		如果误删了文件的话可以用(前提是已经提交到版本库，否则无法恢复)
		~$:git checkout -- file

	远程库
	~$:git remote add origin git@github.com:mxxhcm/testgit.git
	~$:git remote add origin https://github.com/github用户名/github仓库.git
	~$:git push -u origin master 把本地的master库和远程的master库连接起来，
				以后推送或者拉取就会简单
		输入github账号和密码
	　推送
	~$:git push origin master
	　拉取
	~$:git pull	

	还可以直接克隆一个已有的仓库，
	~$:git clone https://github.com/github用户名/github仓库.git
	

	分支管理
	~$:git branch dev 创建分支
	~$:git checkout dev 切换分支
	~$:git branch 查看当前分支
	~$:git merge dev 将分支合并
	~$:git brancd -d dev 删除分支
	
