---
layout: post
title: 详解ServerBootstrap
categories: Netty
description: Netty的学习
keywords: Netty,basic
---

ServerBootstrap负责初始化netty服务器，并且开始监听端口的socket请求。

![ServerBootstrap-1](/images/posts/netty/ServerBootstrap.png)  
ServerBootstrap用一个ServerSocketChannelFactory来实例化。ServerSocketChannelFactory 有两种选择，一种是NioServerSocketChannelFactory，一种是OioServerSocketChannelFactory。 前者使用NIO，后则使用普通的阻塞式IO。它们都需要两个线程池实例作为参数来初始化，一个是boss线程池，一个是worker线程池。  
ServerBootstrap.bind(int)负责绑定端口，当这个方法执行后，ServerBootstrap就可以接受指定端口上的socket连接了。一个ServerBootstrap可以绑定多个端口。  
##### boss线程和worker线程
ServerBootstrap监听的一个端口对应一个boss线程，它们一一对应。比如你需要netty监听8080端口和9999端口，那么就会有两个boss线程分别负责处理来自两个端口的socket请求。在boss线程接收了socket连接请求后，会产生一个channel（一个打开的socket对应一个打开的channel），并把这个channel交给ServerBootstrap初始化指定的ServerSocketChannelFactory来处理，boss线程则继续处理socket的请求。  
ServerSocketChannelFactory则会从worker线程池中找出一个worker线程来处理这个请求。如果是OioServerSocketChannelFactory的话，那么这个channel上所有的socket消息，从开始到channel（socket）关闭，都只由这个特定的worker来处理，也就是说一个打开的socket对应一个指定的worker线程，这个worker线程在socket没有关闭的情况下，也只能为这个socket处理消息，无法服务其他socket。  
如果是NioServerSocketChannelFactory的话则不然，每个worker可以服务不同的socket或者说channel，worker线程和channel不再有一一对应的关系。显然，NioServerSocketChannelFactory只需要少量活动的worker线程就能很好的处理众多的channel，而OioServerSocketChannelFactory则需要与打开channel等量的worker线程来服务。  
线程是一种资源，所以当netty服务器需要处理长连接的时候，最好选择NioServerSocketChannelFactory，这样可以避免创建大量的worker线程。在用作http服务器的时候，也最好选择NioServerSocketChannelFactory，`因为现代浏览器都会使用http keepalive功能（可以让浏览器的不同http请求共享一个信道），这也是一种长连接`。  
##### worker线程的生命周期
当某个channel有消息到达或者有消息需要写入socket的时候，worker线程就会从线程池中取出一个。在worker线程中，消息会经过设定好的ChannelPipeline处理。ChannelPipeline就是一堆有序的filter，它分为两部分：UpStreamHandler和DownStreamHandler。**Pipeline**  
客户端送入的消息会首先由许多**UpstreamHandler** 依次处理，处理得到的数据送入应用的业务逻辑handler，通常用**SimpleChannelUpstreamHandler** 来实现这一部分。  
`public class` 
  `SimpleChannelUpstreamHandler{ `  

        public void messageReceived(ChannelHandlerContext c,MessageEvent e) throws Exception{    
           //业务逻辑开始掺入  
       }
`}`  
对于Nio当**messageReceived()** 方法执行后，如果没有产生异常，worker线程就执行完毕，它会被线程池回收。业务逻辑handler会通过一些方法，把返回的数据交给指定好顺序的**DownStreamHandler** 处理，处理后的数据如果需要，会被写入channel，进而通过绑定的socket发送给客户端。这个过程是由另外一个线程池中的worker线程来完成的。而对于Oio来说，从始至终，都是由一个指定的worker来处理。
##### 减少worker线程的处理占用时间
worker线程是由netty内部管理，统一调配的一种资源，所以最好应该尽快的让worker线程执行完毕，返回给线程池回收利用。worker线程的大部分时间消耗在**ChannelPipeline** 的各种**handler** 中，而在这些handler中，一般是负责应用程序业务逻辑掺入的那个handler最占时间，它通常是排在最后的**UpStreamHandler**。所以通过把这部分处理内容交给另外一个线程来处理，可以有效的减少worker线程的周期循环时间，一般有两种方法：  

* messageReceived()方法中开启一个新的线程来处理业务逻辑  

  `public void messageReceived``(ChannelHandlerContext ctx, MessageEvent e)   throws Exception{     `
     `    ...    `  
     `    new Thread(...).start();    `
 `}`  
在messageReceived()中开启一个新线程来继续处理业务逻辑，而worker线程在执行完messageReceived()就会结束了。更加优雅的方法是另外构造一个线程池来提交业务逻辑处理任务。 
 
* 利用netty框架自带的ExecutionHandler
![ExecutionHandler-1](/images/posts/netty/ExecutionHandler.png)  
把共享的ExecutionHandle实例放置在业务逻辑handler之前即可，注意ExecutionHandle一定要在不同的pipeline之间共享。它的作用是自动从ExecutionHandle自己管理的一个线程池中拿出一个线程来处理排在它后面的业务逻辑handler。而worker线程在经过ExecutionHandle后就结束了，它会被ChannelFactory的worker线程池所回收。  
它的构造方法是ExecutionHandler(Executor executor) ，很显然executor就是ExecutionHandler内部管理的线程池了。  
##### NIO处理方式

![ChannelPipeline-1](/images/posts/ChannelPipeline.png)  
1、Netty用一个Boss线程去处理客户端的接入，创建Channel；  
2、
