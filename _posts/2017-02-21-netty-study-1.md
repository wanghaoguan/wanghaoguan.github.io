---
layout: post
title: Netty学习笔记（一）
categories: Netty
description: Netty的学习
keywords: Netty,basic
---

### Netty--基本概念

Netty项目是一个提供异步事件驱动网络应用框架和快速开发可维护的高性能高扩展性服务端和客户端协议工具集的成果。  
换句话说，Netty是一个NIO客户端服务端框架，它能使快速而简单的开发像服务端客户端协议的网络应用，极大的简化并流线化了如TCP和UDP套接字服务器开发的网络编程。  

### Netty--能做什么  

开始了解到Netty是因为自己需要开发一个消息推送中间件，基本需求是要实现高负载、大并发。  
现在的消息推送普遍采用的解决方案是“长连接”，对于小容量的应用场景，可以使用单线程模型。  
高负载、大并发的环境下需要考虑多线程模型，即使用一组NIO线程来处理IO操作。在这种模型下，使用一个专门的NIO线程来监听服务端，接收客户端的TCP连接请求，而IO操作由一个线程池负责。这种多线程模型在绝大多数场景下可以满足性能要求，但在一些特殊场景中，使用一个NIO线程负责监听和处理所有的客户端连接会存在性能问题。  
Netty的线程模型类似于一种主从多线程模型，这时服务端用于接收客户端连接的不再是一个单独的的NIO线程，而是一个独立的NIO线程池。它的工作流程是从主线程池中随机选择一个Reactor线程作为Acceptor线程，用于绑定监听端口；一旦Acceptor线程接收到客户端连接请求后就创建新的SocketChannel，将其注册到主线程池的其他Reactor线程上，由其负责链路建立之前的操作（接入认证、IP黑名单过滤、握手等）；链路正式建立后，将SocketChannel从主线程池的Reactor线程的多路复用器上摘除，重新注册到Sub线程池的线程上，用于处理IO的读写操作。

### Netty--基本环境

接下来所介绍的示例程序所需要的最小运行需求：最新版本的Netty(4.0)和JDK1.6及以上。 

### Netty--核心组件  
##### Channels  
一个Channels是Java NIO 的一个基本抽象。它代表了：一个连接到比如硬件设备，文件，网络socket等实体的开放连接，或者是一个能够完成一种或多种譬如读或写等不同IO操作的程序。  
目前，可以把一个Channel想象成一个输入和输出数据的媒介。同样的，它可以被打开或者关闭，连接或者断开。  
#####Callbacks
一个callback就是一个方法，一个提供给另一个的方法的引用。这让另一个方法可以在适当的时候回过头来调用这个callback方法，是用于通知相关方某个操作已经完成最常用的方法之一。
Netty在处理事件时内部使用了callback；当一个callback被触发，事件可以被ChannelHandler的接口实现处理。下面的代码是一个例子：当一个新的连接建立后，ChannelHandler的callback方法channnelActive()会被调用，然后打印一条消息。  
![callback](/images/posts/netty/callback.png)
##### Future
一个Future提供了另一个当操作完成时如何通知应用的方法。Future对象充当了一个存放异步操作结果的占位符角色；它会在将来某个时间完成并且提供对操作结果的访问。  
JDK搭载了接口java.util.concurrent.Future, 但是提供的接口实现只允许你手动检查操作是否已经完成，或者就一直阻塞到操作完成。这非常麻烦，所以Netty提供了它自己的ChannelFuture实现，用于执行异步操作。  
ChannelFuture提供了额外的方法让我们可以注册一个或者多个ChannelFutureListener实例。监听者的callback方法operationComplete()在操作完成时被调用，然后监听者可以查看这个操作是否成功完成。简单来说，ChannelFutureListener提供的通知机制免去了手动检查操作完成情况的麻烦。  
每个Netty输出的IO操作都会返回一个ChannelFuture；`就是说，没有一个操作是阻塞的`。就像我们之前所说的，Netty由下至上都是异步和事件驱动的。  
以下代码展示了如何利用ChannelFutureListener。首先连接到一个远端，然后用connect()返回的ChannelFuture注册一个新的ChannnelFutureListener。当监听器被通知连接建立时，检查状态(1)。如果这个操作成功，写数据到这个Channel；否则从ChannelFuture中读取Throwable。  
![callback-2](/images/posts/netty/运作中的callback.png)  
##### Events和Handlers
Netty用细分的events来通知我们状态的变化或者操作的状况，这让我们可以基于发生的events来触发适当的行为（日志记录、数据传送、流控制、应用逻辑）。  
Netty是一个网络编程框架，所以events按他们和输入或者输出数据流的关系来分类。可能被输入数据或者相关状态触发的events包括：活跃或者停用的连接、读数据、用户events、错误events。而输出event则是会触发将来行为的操作的结果，可能会是：打开或者关闭到远端的连接、写或刷数据到一个socket。  
每一个event都可以被分派到一个用户实现的handler对象的方法。下面展示了一个event如何被一串这样的event handler处理。`可以认为每个handler实例就是一种响应某个具体event的callback`
![events](/images/posts/netty/eventHandler处理.png)  
##### 汇总
Netty的异步编程模型是建立在Future和callbacks概念之上的，在更深一层分派事件到handler方法。这些元素结合起来提供了一个处理环境，让你的应用逻辑可以逐步发展而不用关心网络操作。这一个Netty设计方案的一个关键目标。  
Netty通过引发事件把Selector从应用中抽象出来，省掉了所有原本需要手写的调度代码。在内部，一个EventLoop被分配到每个Channel来处理所有的events，包括注册感兴趣的events、分派events到ChannelHandlers、安排将来的行为。  
EventLoop自己仅由一个线程驱动，这个线程处理一个Channel所有的I/O事件，这个关系在Eventloop的生命周期内不会改变。这个简单强大的设计消除了任何你可能对ChannelHandler同步的顾虑，因此你能够专注于在数据被处理时，提供正确的执行逻辑。