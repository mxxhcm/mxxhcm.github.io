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

## epoll三板斧
### epoll_create
使用epoll_create创建一个eventpoll结构体，每一个epoll对象都有一个eventpoll结构体，用于存放epoll_ctl添加的event，这样子可以识别重复的event，添加到epollpoll的event都会和设备驱动程序建立会调用关系，当相应的事件发生时调用相应的回调方法，将发生的event添加到一个rdlist双向链表中。
每次轮询只要查看rdlist是否为空即可。

在2.6.8版本之前，epoll的底层实现是hash，所以需要指定一个size大小，从2.6.8版本开始，epoll的底层实现是红黑树，size大小被忽略，但是必须比零大。
epoll_create返回一个文件描述符，用完之后要调用close。

### epoll_ctl
epoll_ctl用于注册事件。```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
epoll_ctl有四个参数：
- int类型文件描述符，epoll_create创建的epoll对应的描述符，指定操作的epoll实例。
- 指向对描述符指定的epoll实例的操作，可取值为EPOLL_CTL_ADD, EPOLL_CTL_MOD, EPOLL_DEL。
- int类型的文件描述符，指定要监听的某个描述符。
- struct epoll_event类型的事件，指定我们对第三个参数指定的描述符感兴趣的事件。``` struct epoll_event{
    uint32_t events;    // epoll事件
    epoll_data_t data;  //用户数据变量
};
```
其中events可以是
EPOLLIN, 对应的描述符可读。
EPOLLOUT, 对应的描述符可写。
EPOLLPRI，对应的描述符有紧急数据可读。
EPOLLERR, 对应的描述符发生错误。
EPOLLET，设置为ET模式。
EPOLLHUP，对应的描述符被挂断。
EPOLLRDHUP，。
等

### epoll_wait
epoll_wait监听使用epl_ctl添加的事件。```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```
其中有四个参数：
- int类型的文件描述符，指定等待的epoll实例。
- struct epoll_event类型的指针，不能是空指针，内核会把已经就绪的文件描述符复制到这个数组中。
- maxevents指定最多可以返回的事件数，必须大于0。
- timeout指定阻塞的时间。

## LT模式和ET模式
epoll有两个工作模式，一个是edge-triggered，另一个是level-triggered。为了区别它们的不同，给出一个场景：
1. epoll实例中添加了一个pipe的读端。
2. pipe的写段向其中写入了2KB的数据。
3. epoll_wait会返回这个pipe的描述符作为一个准备好的描述符。
4. pipe的读端从pipe中读了1KB的数据。
5. 对epoll_wait的调用已经完成。

如果添加这个描述符使用的是EPOLLET模式，那么对于epoll_wait的调用会继续阻塞，尽管它的输入缓冲区中还有数据。同时pipe的写端可能在等着对它已经发送数据的一个回复。**这是因为ET模式只会在它监视的fd发生变化时才会报告event。**
在上面的例子中，因为2中的写完成会产生一个event，然后这个evenet在3中被使用。因为4中的读没有把buffer中的所有数据读完，对于5中的epoll_wait可能会永远阻塞。

为了避免处理多个文件描述符的永远阻塞，建议使用epollET模式的方式，这两个应该是一块设置的：
1. 使用nonblocking file descriptors，并且
2. 只有在read或者write返回一个EAGAIN时，再调用epoll_wait等待一个event。或者是读写的字节数小于指定的字节数时。如果是阻塞描述符的话，这样子就会阻塞在最后一次读写调用。而导致即使有其他的描述符可以操作时，因为阻塞在这个调用中，就不能执行其他的读写操作。

而epoll的LT模式，比poll快，可以用在任何出现在poll出现的地方。

## epoll和select
什么时候使用select而不使用epoll，epoll适用于连接特别多但是活跃连接少的场景，而select适用于连接多活跃连接也多的场景。

## 常见问题
0. 那么如果一个描述符一直可读，会不会导致其他进程饿死？？？怎么办？[3]
可以设置一个结构体，将fd和一个标志位联系起来，epoll_wait事件触发后，将相应的结构体的标志位置为ready，然后轮询这个列表中的描述符（刚开始一直没向明白，后来才意识到，列表中ready的描述符每一个都能被处理）。
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
2.https://www.cnblogs.com/tianzeng/p/9997432.html
3.https://stackoverflow.com/questions/21111003/epoll-tcp-edge-triggered-necessity-of-last-read2-call
