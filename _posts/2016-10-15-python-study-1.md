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
安装结束后，打开cmd控制台，输入python，如果出现以下画面，则证明python已成功安装。

![python-install](/images/posts/python/install.png)

### 第一个Python程序

打开cmd，进入python交互环境中，在`>>>`提示符下直接输入代码，就可以立刻得到执行结果。测试用例：
`>>>600 + 66`，回车得到输出`666`。  
如果需要打印出指定的文字，可以直接使用`print`语句，输出的语句需要用单引号或双引号括起来。测试用例：`>>>print 'hello,world'`，回车得到输出`hello,world`。  
退出python可以直接关掉控制台，也可以用`exit()`来退出。

![python-test-1](/images/posts/python/test-1.png)



