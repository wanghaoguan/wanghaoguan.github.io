---
layout: post
title: Netty学习笔记（二）
categories: Netty
description: Netty的学习
keywords: Netty,basic
---

今天的主要任务是：  
* 创建开发环境  
* 编写一个Echo服务器和客户端  
* 编译测试应用  
### 创建开发环境
为了完成项目，需要用到的工具是JDK和Apache Maven，它们都是免费下载的。  
##### 获取和安装Java开发套件
参照Java官方文档，安装配置完成后，在命令行中敲如下命令：`javac -version`
##### 下载和安装一个IDE
下面几个是最广泛使用的Java IDEs，都是免费的：  
* Eclipse—www.eclipse.org  
* NetBeans—www.netbeans.org  
* Intellij Idea Community Edition—www.jetbrains.com
##### 下载和安装Apache Maven
Maven是一个广泛使用的项目构建管理工具，由Apache软件公司开发。Eclipse和NetBeans都内嵌了Maven安装，可以立即为我们所用。另外如果你工作在一个有自己Maven仓库的开发环境中，你的管理员可能已经有了一个Maven安装包以便预先配置到这个仓库。  
最新的Maven版本是3.3.3。你可以从http://maven.apache.org/download.cgi为你的系统下载合适的tar.gz或者zip文件。安装很简单：提取压缩包的内容到你选择的目录（我们称为<install_dir>）。目录<install_dir>\apache-maven-3.3.3就创建好了。  

关于Java环境：  
* 把环境变量M2_HOME配置成<install_dir>\apache-maven-3.3.3  
* 把%M2_HOME%\bin（或者在Linux上是${M2_HOME}/bin）加入你的执行路径。

### Netty服务器/客户端概览
![demo-1](/images/posts/netty/demo-1-1.png)  
这个图展示的是同时连接到一个服务器的多个客户端。理论上，可支持的客户端数量是受可支配的系统资源限制的。  
客户端和服务器之间的交互非常简单; 客户端建立一个连接后，它送一条或者多条消息到服务器，然后服务器再将该消息送回客户端。虽然这个交互看起来不那么实用，但却是一个典型的客户端/服务器系统请求-响应交互过程的示例。

### 开发服务器端
所有的Netty服务器都需要：  
* 至少一个ChannelHandler--实现服务器如何处理从客户端收到的数据  
* Bootstrapping--这是配置服务器的启动代码。至少要做的是，他把服务器绑到一个可以监听连接请求的端口上。

###### ChannelHandlers
ChannelHandler，它是一组接口的父类，这些接口的实现类接收并且响应事件通知。在Netty应用中，所有的数据处理逻辑都是包含在这些核心抽象(core abstractions)的实现类里的。  

服务器会响应收到的消息，所以它需要实现接口ChannelInboundHandler，这个接口定义了作用于输入事件的方法。这个简单的应用只需要这个接口的一些方法，所以用子类ChannelInboundHandlerAdapter就足够了，这个子类提供了ChannelInboundHandler的缺省实现。

我们感兴趣的是下面几个方法：

* channelRead()—每次收到消息时被调用
* channelReadComplete()—用来通知handler上一个ChannelRead()是被这批消息中的最后一个消息调用
* exceptionCaught()—在读操作异常被抛出时被调用  
![demo-1](/images/posts/netty/demo-1-2.png)  

##### Bootstrapping服务器
