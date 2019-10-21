---
title: github .gitignore
date: 2019-04-29 16:03:21
tags:
- git
categories: git
mathjax: true
---


## gitignore介绍
.gitignore是一个隐藏文件，用来指定push的时候忽略哪些文件和文件夹。
比如忽略所有的\_\_pychche\_\_文件夹
\*\*/\_\_pycache\_\_/
这里一定要加上/否则就会把它当做一个文件来处理

## 删除git服务器上已有的在.gitignore的文件
但是.gitignore对于已经提交到git服务器的文件是无法删掉的，它在提交时只能忽略本地尚未同步到服务器的gitignore中出现的文件。
拿.idea举个例子。
在最开始的时候，没有写.gitignore文件，就把所有的python文件上传到了git，包括.idea文件，这时候，可以先在本地把.idea文件删了，然后commit一下，就把git上的.idea文件删了。这时候写.gitignore文件，以后就不会提交.idea文件了。
执行以下命令：
``` shell
find . -name '**idea' | xargs git rm -rf
# 或者find . -name '*idea' | xargs git rm -rf
git add .
git commit -m "deleta *idea"
git push
```
这里首先使用find找到当前目录下所有.idea文件夹，然后使用管道命令将其删除，再提交到git。
接下来在.gitignore文件中添加一行：
\*\*/.idea/
然后再次提交到git的时候就不会同步.idea文件了。

## 加注释
.gitignore文件的注释使用#号开头即可。

## 参考文献
1.https://segmentfault.com/q/1010000000720031

