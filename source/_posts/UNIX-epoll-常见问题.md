---
title: UNIX epoll 常见问题
date: 2020-03-17 09:16:44
tags:
 - UNIX
categories: UNIX
---

## select和epoll区别
1. select单个进程能够监视的文件描述符的数量存在最大限制，通常是1024，不过是可以改的，而poll没有这个限制（使用链表），epoll是无限制的。
2. 传参。select和poll需要复制大量的file descriptors。
3. 返回值。返回的值所有fd的集合，需要遍历才知道哪几个fd发生了event。
4. 如果在循环中使用select，每次都要重新设置。

使用epoll_create创建一个eventpoll结构体，每一个epoll对象都有一个eventpoll结构体，用于存放epoll_crt添加的event，这样子可以识别重复的event，添加到epollpoll的event都会和设备驱动程序建立会调用关系，当相应的事件发生时调用相应的回调方法，将发生的event添加到一个rdlist双向链表中。
每次轮询只要查看rdlist是否为空即可。


## LT模式和ET模式
epoll有两个工作模式，一个是edge-triggered，另一个是level-triggered。为了区别它们的不同，给出以下一个场景：
1. epoll实例中添加了一个pipe的读端。
2. pipe的写段向其中写入了2KB的数据。
3. epoll_wait会返回这个pipe的描述符作为一个准备好的描述符。
4. pipe的读端从pipe中读了1KB的数据。
5. 对epoll_wait的调用已经完成。

如果添加这个描述符使用的是EPOLLET模式，那么对于epoll_wait的调用会继续阻塞，尽管它的输入缓冲区中还有数据。同时pipe的写端可能在等着对它已经发送数据的一个回复。这是因为ET模式只会在它监视的fd发生变化时才会报告event。
在上面的例子中，因为2中的写完成会产生一个event，然后这个evenet在3中被使用。因为4中的读没有把buffer中的所有数据读完，对于5中的epoll_wait可能会永远阻塞。
为了避免处理多个文件描述符的永远阻塞，建议使用epoll的ET模式时：
1. 使用nonblocking file descriptors。
2. 只有在read或者write返回一个EAGAIN时，再调用epoll_wait等待一个event。

而epoll的LT模式，比poll快，可以用在任何出现在poll出现的地方。

## 常见问题
1. epoll中区分不同file descriptors的key是什么？key是file descriptor number和open file description的组合。
2. 同一个epoll实例注册同一个file descriptor会发生什么？会获得EEXIST。但是可以添加一个dup复制的file descriptor到相同的epoll instance。
3. 两个epoll实例可以等待相同的fd吗？可以的，evenets将会通知这两个epoll实例。
4. epoll fd自己是不是可以被poll/epoll？是的！
5. 将epoll fd放在它自己的fd sets中？epoll_ctl会失败，产生EINVAL。但是可以把它添加到其他epoll fd set中。
6. 可不可以通过UNIX域套接字发送epoll文件描述符，可以。但是最好不要这样，因为其他进程可能没有epoll监听的描述符。
7. 关闭一个fd， epoll会不会自动把它从epoll sets中移除。会，但是注意，文件描述符只是file descriptor的一个引用，当一个file descriptor被复制的话，直到所有的文件描述符都被关闭，才会被移除。
8. 超过一个event发生，它们会一块reported。
9. 一个fd上的操作是否会影响已经collected但是还没有reported的events。
10. 使用ET模式的话，是否必须连续到EAGAIN？面向报文的需要，而面向流的，需要看target file descriptor中还有多少数据。


## 参考文献
1.《UNIX程序员手册》
