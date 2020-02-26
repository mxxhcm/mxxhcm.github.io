---
title: UNIX system function
date: 2020-02-20 16:42:45
tags:
 - UNIX
categories: UNIX
---


## system
ISO C定义了`system`函数，它用来执行一个shell命令，对系统的依赖性很强。`system`库函数调用`fork`创建一个子进程使用`execl`执行参数`system`指定的shell命令。相当于:```c
execl("/bin/sh", "sh", "-c", command, (char *) 0);
```
POSIX.1包括了`system`接口，扩展了ISO C定义，描述了`system`在POSIX.1环境中的运行行为。
```c
#include <stdlib.h>

int system(const char *command);
```

## 属性
`system`在其实现中调用了`fork`, `exec`和`waitpid`，可能有以下的返回值：
1. 如果`command`是一个空指针，当前系统是有可用的shell时，`system`返回非0值，否则返回0。在UNIX的各个实现中，一定提供了shell，当`command`是空指针时，总是返回非零值。
2. `fork`失败或者`waitpid`返回处`EINTR`之外的出错，返回-1，设置errno。
3. 如果`exec`失败，表示不能执行shell，返回值如同shell执行了exit(127)一样
4. 所有三个函数都成功，`system`的返回值和在shell中执行相应命令的的termination status一样。

## 可能实现
其中一种`system`的可能实现如下所示：```c
int system(const char *cmdstring)
{
    pid_t pid;
    int status;

    if(cmdstring == NULL)
        return 1;
    
    if ((pid = fork()) < 0)
    {
        status = -1;
    }
    else if(pid == 0)
    {
        execl("/bin/sh", "sh",  "-c", cmdstring, (char*)0);
        _exit(127);
    }
    else
    {
        while(waitpid(pid, &status, 0) < 0)
        {

            if(errno != EINTR)
            {
                status = -1;
                break;
            }
        }
    }

    return status;
}
```
使用`system`而不是直接使用`fork`和`exec`的好处是：`system`进行了各种所需要的各种出错处理和信号处理。但是早期的系统中，没有`waitpid`函数，于是父进程使用下列语句等待子进程结束：``` c
while((lastpid = wait(&status) != pid && lastpid !=-1)
    ;
```
如果在调用`system`之前产生了其它子进程，如果这些子进程在`system`产生的子进程之前结束，那么上面的代码会将这些提前结束的子进程的进程ID和termination都给丢弃。
**system函数还有可能会出现漏洞。如果设置了set UID或者set GID位的程序执行`system`，那么这个进程的高级别权限可能会保持下来（现代的系统都解决了这个问题）。如果一个进程正在以特殊的权限(set UID和set GID)运行，它又想生成另一个进程执行另一个程序，它应该直接使用`fork`和`exec`，而且在`fork`之后，`exec`之前要改回普通权限，set UID和set GID程序绝不应该调用`system`函数。**


## 参考文献
1.《APUE》第三版

