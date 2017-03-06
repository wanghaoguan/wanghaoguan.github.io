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

   




