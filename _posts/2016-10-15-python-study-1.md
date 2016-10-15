---
layout: post
title: Python基础（一）
categories: Python
description: python的学习
keywords: python,basic
---

Python是是一种面向对象的解释型计算机程序设计语言。Python的设计目标之一是让代码具备高度的可阅读性。它设计时尽量使用其它语言经常使用的标点符号和英文单字，让代码看起来整洁美观。它不像其他的静态语言如C、Pascal那样需要重复书写声明语句，也不像它们的语法那样经常有特殊情况和惊喜。

Python的特点可以总结为：

优点：开发快，功能强大，程序易读。

缺点：运行速度慢，代码不能加密。

参考：<http://baike.baidu.com/link?url=Rm-EDfcyzMvQTHkVFaRSikRzh1y8ew5OlMKlcKcIZL4xakSoUd5wInePRtnGz3NtSdxyNBLI-Utbp-ollqsdFq>

### 在win7安装Python

首先，需要从从Python的官方网站<http://www.python.org>下载python,然后运行下载的msi安装包，安装所有组件，安装时要选择`Add python.exe to Path`将安装路径添加到系统`Path`环境变量。然后一路`next`到安装程序完成。  
![python-install-1](/images/posts/python/install-1.png)

安装结束后，打开cmd控制台，输入python，如果出现以下画面，则证明python已成功安装。

![python-install](/images/posts/python/install.png)

### 第一个Python程序

打开cmd，进入python交互环境中，在`>>>`提示符下直接输入代码，就可以立刻得到执行结果。测试用例：
`>>>600 + 66`，回车得到输出`666`。  
如果需要打印出指定的文字，可以直接使用`print`语句，输出的语句需要用单引号或双引号括起来。测试用例：`>>>print 'hello,world'`，回车得到输出`hello,world`。  
退出python可以直接关掉控制台，也可以用`exit()`来退出。

![python-test-1](/images/posts/python/test-1.png)

### 使用文本编辑器

在控制台用命令行写程序，可以立即得到结果，但程序无法保存。所以我们实际开发中，需要使用文本编辑器来编辑代码，将代码保存为文件就可以反复运行。  

我在这里使用的是Sublime Text，这款编辑器具有漂亮的用户界面和强大的功能,支持丰富的插件管理。Sublime Text 是一个跨平台的编辑器，同时支持Windows、Linux、Mac OS X等操作系统。关于这款编辑器的详细情况可以参考：
<http://baike.baidu.com/link?url=1pKMEKuBtPZ_eI4S49YXoiKUIpwbtXyP1fv7-WZmhZ96nzFnP96ciS9jjHLRjmj86yDMahuvwJ-rR1bTQDKWY9uaq8ddylAK2VmZiTw2j2a>

在Sublime Text中新建文件，输入代码 `print 'hello,world'`，并保存为`hello.py`。

![python-edit-1](/images/posts/python/edit-1.png)

然后打开命令行窗口，切换到当前目录，就可以运行这个程序了。

![python-test-2](/images/posts/python/test-2.png)

Python的交互模式和直接运行.py文件有什么区别呢？

直接输入python进入交互模式，相当于启动了Python解释器，但是等待你一行一行地输入源代码，每输入一行就执行一行。

直接运行.py文件相当于启动了Python解释器，然后一次性把.py文件的源代码给执行了，你是没有机会输入源代码的。

用Python开发程序，完全可以一边在文本编辑器里写代码，一边开一个交互式命令窗口，在写代码的过程中，把部分代码粘到命令行去验证，事半功倍！

