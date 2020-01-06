---
title: UNIX signals
date: 2019-12-05 10:20:16
tags:
 - UNIX
 - signal
categories: UNIX
---

## Program Error Signals
当操作系统或者计算机本身检测到一个严重的程序错误时，它就会产生program error signals。一般来说，这些signals都表示你的程序出现了很严重的问题，没有办法继续进行计算了。
一些程序为了在进程终止前进行清理，会建立handle program。这个handler结束的时候应该指定该signal的默认动作或者重新raise这个signal，这样子和原来没有建立signal处理程序的时候相比，仅仅多了清理的一部分，其他还和原来一样。一般来说，这些signal的默认动作都是终止进程。如果不是`raise`或者`kill`发出的这些signals，选择block或者忽略这些singals或者建立handlers让它们正常返回，程序可能会崩溃。
当某一个program error signals终止一个进程的时候，它会写一个core dump文件记录进程终止状态。这个core dump文件的名字叫做core，保存在进程终止时所在目录。（在ubuntu中，没有保存文件，而是直接输出了出来[3]）

### `SIGFPE`
### `SIGILL`
### `SIGBUS`
### `SIGSEGV`
### `SIGABRT`
### `SIGTRAP`
### `SIGEMT`

## Terminal Signals
这一小节介绍的signal都是由于terminate一个进程的。它们之间的区别在于使用目的不同，并且程序可能对不同的signals有不同的处理方法。
处理这些signals的目的是让进程在terminate之前能够进行合适的清理。比如，删除临时文件等等。

### `SIGTEM`
这个signal可以被blocked, handled和忽略，shell命令的`kill`默认会产生`SIGTERM` signal。
> It is the normal way to politely ask a program to terminate.

### `SIGINT`
`SIGINT`是程序中断singal，当用户输入INTR字符（通常是C-c）时中断。

### `SIGQUIT`
`SIGQUIT`和`SIGINT`很像，会中断一个进程，但是它被QUIT（通常是C-\）控制，并且当它terminate一个进程时，它会产生一个core dump，就像一个程序出错信号一样。
在处理`SIGQUI`的时候，最好不进行某些cleanups。比如，如果程序创建临时文件，处理其他termination时，最好把临时文件给删除了，但是对于`SIGQUIT`来说，最好不把它们给删了，因为用户可以用它们查找原因。

### `SIGKILL`
`SIGKILL` signal用于立刻终止程序。它不能被handled，也不能被忽略，因此是致命的。同样的也不能被block。
这个singal通常是显式请求。因为他不能被handled，所以一般把它作为最后一个选择，在尝试了C-c或者`SIGTERM`不起作用之后，最后再使用`SIGKILL`。

### `SIGHUP`
如果终端借口检测到一个连接断开（可能因为网络或者电话连接断了），就将这个signal报告和这个终端相关的控制进程。


### 其他
> The default behavior of SIGINT, SIGTERM, SIGQUIT and SIGHUP is to kill the program. However applications are allowed to install a handler for these signals. So the actual behavior of applications when they receive these signals is a matter of convention (which each application may or may not follow), not of system design.
> SIGINT is the “weakest” of the lot. Its conventional meaning is “stop what you're doing right now and wait for further user input”. It's the signal generated by Ctrl+C in a terminal. Non-interactive programs generally treat it like SIGTERM.
> SIGTERM is the “normal” kill signal. It tells the application to exit cleanly. The application might take time to save its state, to free resources such as temporary files that would otherwise stay behind, etc. An application that doesn't want to be interrupted during a critical application might ignore SIGTERM for a while.
> SIGQUIT is the harshest of the ignorable signals. It's meant to be used when an application is misbehaving and needs to be killed now, and by default it traditionally left a core dump file (modern systems where most users wouldn't know what a core file is tend to not produce them by default). Applications can set a handler but should do very little (in particular not save any state) because the intent of SIGQUIT is to be used when something is seriously wrong.
> SIGKILL never fails to kill a running process, that's the point. Other signals exist to give the application a chance to react.
> SIGHUP is about the same as SIGTERM in terms of harshness, but it has a specific role because it's automatically sent to applications running in a terminal when the user disconnects from that terminal (etymologically, because the user was connecting via a telephone line and the modem hung up). SIGHUP is often involuntary, unlike SIGTERM which has to be sent explicitly, so applications should try to save their state on a SIGHUP. SIGHUP also has a completely different conventional meaning for non-user-facing applications (daemons), which is to reload their configuration file.

## Job Control Signals
### `SIGCLD`

## 参考文献
1.https://unix.stackexchange.com/questions/251195/difference-between-less-violent-kill-signal-hup-1-int-2-and-term-15
2.https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html#Termination-Signals
3.https://stackoverflow.com/questions/2065912/core-dumped-but-core-file-is-not-in-the-current-directory