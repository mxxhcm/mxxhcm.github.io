---
title: linux python调用shell脚本并将输出重定向到文件
date: 2019-06-03 20:11:38
tags:
- python
- linux
- shell
- 工具
- 常见问题
categories: 
- [python]
- [linux]
---

## python执行shell脚本并且重定向输出到文件
目的：有一些shell脚本的参数需要调整，在shell中处理有些麻烦，就用python控制参数，然后调用shell，问题就是如何将shell脚本的输出进行重定向。最开始我想直接用python调用终端中shell重定向的语法，我用的是os.system(command)，command包含重定向的命令，在实践中证明是不可行的。为什么？？？留待解决。

找到的解决方案是调用subprocess包，将要执行的命令存入一个list，将这个list当做参数传入，获得返回值，进行文件读写。这里拿ls命令举个例子，注意一点是，包含重定向语句的ls命令使用os.system()也能重定向成功，我使用的一个命令就不行了。
### call
``` python
subprocess.call(['ls', '-l', '.']) # 直接将程序执行结果输出，没有返回值。
```
### Popen
``` python
results = subprocess.Popen(['ls', '-l', '.'], 
            stdout=subprocess.PIPE, 
            stderr=subprocess.STDOUT)
stdout, stderr = results.communicate() 
res = stdout.decode('utf-8') # 利用res将结果输出到文件
```
### run
``` python
result = subprocess.run(['ls', '-l'], stdout=subprocess.PIPE) 
res = result.stdout.decode('utf-8')  # 利用res将结果输出到文件
```

## 参考文献
1.https://stackoverflow.com/questions/4760215/running-shell-command-and-capturing-the-output
2.https://linuxhandbook.com/execute-shell-command-python/
