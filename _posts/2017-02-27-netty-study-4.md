---
layout: post
title: Linux环境下几种IO解析
categories: Netty
description: Netty的学习
keywords: Netty,basic
---

对于一个Network IO，它会涉及到两个系统对象，一个是调用这个IO的process(or thread),另一个就是系统内核。当一个read操作发生时，它会经历两个阶段：  
1、等待数据准备  
2、将数据从内核拷贝到进程中  
接下来提到的几种IO Model在这两个阶段各有不同的情况。  

##### Blocking IO 
在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概如下：  
![BlockingIO-1](/images/posts/netty/BlockingIO.png)  
当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候kernel就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。  
所以，blocking IO的特点就是在IO执行的两个阶段都被block了。
##### Non-Blocking IO
linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程如下：  
![non-blockingIO-1](/images/posts/netty/non-blockingIO.png)   
当用户进程发出read操作时，如果kernel中的数据还没准备好，那么他并不会block用户进程，而是立刻返回一个error。从用户进程角度讲，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，他就知道数据还没准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。  
所以，用户进程其实是需要不断的主动询问kernel数据准备好了没有。
##### IO multiplexing
IO multiplexing这个词可能有点陌生，但是如果我说select，epoll，大概就都能明白了。有些地方也称这种IO方式为event driven IO。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。它的流程如图：  
![multiplexingIO-1](/images/posts/netty/multiplexingIO.png)  
当用户进程调用了select，那么整个进程会被block，而同时，kerne会监视所有select负责的socket，当任何一个socket中的数据准备好了，selec就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。这里需要使用两个system call（select和recvfrom），而blocking IO只调用了一个system call(recvfrom)。但是，用select的优势在于它可以同时处理多个connection。(select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。)  
在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。
##### Asynchronous IO

![AsynchronousIO-1](/images/posts/netty/AsynchronousIO.png)  
用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

**经过上面的介绍，会发现non-blocking IO和asynchronous IO的区别还是很明显的。在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。**