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

1、Netty用一个Boss线程去处理客户端的接入，创建Channel；  
2、从work线程池拿出一个work线程交给boss创建好的Channel实例；  
3、work线程进行数据读入(读到ChannelBuffer)；  
4、接着触发相应的事件传递给ChannelPipeline进行业务处理（ChannelPipeline中包含一系列用户自定义的ChannelHandler组成的链）；  
![ChannelPipeline-1](/images/posts/ChannelPipeline.png)  
有一点要注意，执行整个ChannelHandler链这个过程是串行的，如果业务逻辑比较耗时，会导致work线程长时间被占用得不到释放，最终影响整个服务端的并发处理能力，所以我们一般通过ExecutionHandler线程池来异步处理ChannelHandler调用链，使得work线程经过ExecutionHandler时得到释放。  

Netty为ExecutionHandler提供了两种可选的线程池模型：  

* MemoryAwareThreadPoolExecutor
通过对线程池内存的使用控制，可控制Executor中待处理任务的上限（超过上限时，后续进来的任务将被阻塞），并可控制单个Channel待处理任务的上限，防止内存溢出错误；  
处理同一个Channel的事件，是串行的方式执行的，但是同一个Channel的多个事件，可能会分布到线程中池中的多个线程去处理，不同的Channel事件可以并发处理，互相并不影响。

* OrderedMemoryAwareThreadPoolExecutor
是 1）的子类。除了MemoryAwareThreadPoolExecutor 的功能之外，它还可以保证同一Channel中处理的事件流的顺序性，这主要是控制事件在异步处理模式下可能出现的错误的事件顺序，但它并不保证同一Channel中的事件都在一个线程中执行，也没必要保证这个。  
同一个Channel的事件，并不保证处理顺序，可能一个线程先处理了Channel A (Event 3)，然后另一个线程才处理Channel A (Event 2)，如果业务不要求保证事件的处理顺序，我认为还是尽量使用MemoryAwareThreadPoolExecutor比较好。  

**Netty采用标准的SEDA（Staged Event-Driven Architecture）架构**  
SEDA的核心思想是把一个请求处理过程分成几个stage，不同资源消耗的stage使用不同数量的线程来处理，stage间使用事件驱动的异步通信模式。更进一步，在每个stage中可以动态配置自己的线程数，在超载时降级运行或拒绝服务。  
Netty所设计的事件类型，代表了网络交互的各个阶段，每个阶段发生时，会触发相应的事件并交给ChannelPipeline进行处理。事件处理都是通过Channels类中的静态方法调用开始的。  

**Netty将网络事件分为两种类型：**  
1、Upstream：上行，主要是由网络底层反馈给Netty的，比如messageReceived、channelConnected  
2、Downstream：下行，框架自己发起的，比如bind、write、connect等

**Netty的ChannelHandler分为3种类型：**  
1、只处理Upstream事件：实现ChannelUpstreamHandler接口  
2、只处理Downstream事件：实现ChannelDownstreamHandler接口  
3、同时处理Upstream和Downstream事件：同时实现ChannelUpstreamHandler和ChannelDownstreamHandler接口  

**Channels中事件流转静态方法：**  
1. fireChannelOpen  
2. fireChannelBound  
3. fireChannelConnected  
4. fireMessageReceived  
5. fireWriteCompleteLater  
6. fireWriteComplete  
7. fireChannelInterestChangedLater  
8. fireChannelDisconnectedLater  
9. fireChannelDisconnected  
10. fireChannelUnboundLater  
11. fireChannelUnbound  
12. fireChannelClosedLater  
13. fireChannelClosed  
14. fireExceptionCaughtLater  
15. fireExceptionCaught   
16. fireChildChannelStateChanged  

ChannelPipeline 维持所有ChannelHandler的有序链表，当有Upstresam或Downstream网络事件发生时，调用匹配事件类型的 ChannelHandler来处理。ChannelHandler自身可以控制是否要流转到调用链中的下一个 ChannelHandler（ctx.sendUpstream(e)或者ctx.sendDownstream(e)）,这一样有一个好处，比如业务 数据Decoder出现非法数据时不必继续流转到下一个ChannelHandler。