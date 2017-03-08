---
layout: post
title: ChannelHandler和ChannelPipeline
categories: Netty
description: Netty的学习
keywords: Netty,basic
---

### ChannelHandler家族
##### Channel的生命周期
接口Channel定义了一个简单强大，并且和ChannelInboundHandler API密切相关的状态模型。四种Channel状态如下所示：

**Channel生命周期状态**  
![Channel生命周期状态-1](/images/posts/netty/Channel生命周期状态.png)   
一个Channel随着状态发生变化，相应的event产生。这些event被转发到channelPipeline中的ChannelHandler来采取相应的操作。  
![Channel状态模型-1](/images/posts/netty/Channel状态模型.png)   

##### ChannelHandler的生命周期
接口ChannelHandler定义的生命周期相关的操作如下表。这些操作在ChannelHandler被添加到一个ChannelPipeline，或者从一个ChannelPipeline中移除时被调用。这里的每个方法都包含一个ChannelHandlerContext作为输入参数。  
**ChannelHandler生命周期方法**  
![ChannelHandler生命周期方法-1](/images/posts/netty/ChannelHandler生命周期方法.png )  
Netty定义了下面两个重要的ChannelHandler子接口  

* ChannelInBoundHandler——处理输入数据和所有类型的状态变化  
* ChannelOutBoundHnadler——处理输出数据，可以拦截所有操作  

##### ChannelInbouundHandler接口
下面列出了接口ChannelInboundHandler生命周期相关的方法。当收到数据，或者相关Channel的状态改变时，这些方法被调用。这些方法和Channel的生命周期密切相关。  
**ChannelInboundHandler方法** 
![ChannelInboundHandler方法-1](/images/posts/netty/ChannelInboundHandler方法.png)   
当一个ChannnelInboundHandler实现类重写channelRead()方法时，它要负责释放ByteBuf相关的内存。Netty提供了一工具方法ReferenceCountUtil.release()。  
![ChannnelInboundHandler-1](/images/posts/netty/ChannnelInboundHandler.png)  
   
对未释放的资源，Netty会打印一个警告级别的日志消息。但是用这种方式管理资源有些麻烦。一个更简单的方法是用SimpleChannelInboundHandler。下面说明了SimpleChannelInboundHandler的用法。  
![SimpleChannelInboundHandler-1](/images/posts/netty/SimpleChannelInboundHandler.png)  
##### ChannelOutboundHandler接口
输出的操作和数据由ChannelOutboundHandler处理。它的方法可以被Channel，ChannelPipeline和ChannelHandlerContext调用。
ChannelOutboundHandler有一个强大的功能，可以按需推迟一个操作。比如，如果写数据到远端被暂停，你可以推迟Push操作。下面列出了所有ChannelOutboundHandler自己定义的方法。  
![ChannelOutboundHandler-1](/images/posts/netty/ChannelOutboundHandler.png)  
##### ChannelHandler适配器类
ChannelInboundHandlerAdapter和ChannelOutboundHandlerAdapter这两个适配器类分别提供了ChannelInboundHandler和ChannelOutboundHandler的基本实现。它们继承了共同的父接口ChannelHandler的方法，扩展了抽象类ChannelHandlerAdapter。类层级关系如下图：  
![ChannelHandlerAdapter类层级关系-1](/images/posts/netty/ChannelHandlerAdapter类层级关系.png)  

ChannelHandlerAdapter还提供了一个工具方法isSharable()。如果类实现带有@Sharable注释，那么这个方法就会返回true，意味着这个对象可以被添加到多个ChannelPipeline中。
### ChannelPipeline接口
可以把一个ChannelPipeline看成是一串ChannelHandle实例，拦截穿过Channel的输入输出event。每个新创建的Channel都会分配一个新的ChannelPipeline。这个关系是恒定的；Channel不可以换别的ChannelPipeline，也不可以解除当前分配的ChannelPipeline。在Netty组件的整个生命周期中这个关系是固定的。  
根据来源，一个event可以被一个ChannelInboundHandler或者ChannelOutboundHandler处理。接下来，通过调用ChannelHandlerContext的方法，它会被转发到下一个同类型的handler。  
**ChannelHandlerContext**让一个ChanneHandler可以和它的ChannelPipeline和其他handler的交互。通过ChannelHandlerContext，一个handler可以通过ChannelPipeline中的下一个ChannelHandler。 
##### 修改ChannelPipeline
改动ChannelPipeline的ChannelHandler方法如下表：  
![Pipe方法-1](/images/posts/netty/Pipe方法.png)  
下面的代码说明了这些方法该怎么使用：  
![Pipe方法-2](/images/posts/netty/Pipe方法-2.png)  
通常ChannelPipeline中的每个ChannelHandler通过它的EventLoop（IO线程）来处理传给他的event。不阻塞这个线程是非常重要的，因为阻塞有可能会对整个IO处理造成负面影响。  


