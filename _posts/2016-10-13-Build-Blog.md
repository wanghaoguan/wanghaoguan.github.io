---
layout: post
title: 使用Github和Jekyll搭建个人博客
categories: Github
description: 一直都有搭建一个个人博客的想法，刚好近日空闲时间比较多，于是用了一天的时间完成了第一版的个人独立博客的搭建
keywords: github, jekyll
---


搭建个人博客的想法已经出现过很多次，养成写博客的习惯，可以让自己及时总结学过的知识，还可以增强我们做事的条理性和逻辑性，之前由于各种事情和自己拖延症的毛病，迟迟没有进行。今天终于用了一天的时间基本完成了独立博客的搭建。

**目录**

* TOC
{:toc}

### 利用GitHub Pages创建一个初始blog

1. 如果你没有接触过github，你需要先注册一个	github的帐号，并登录    
   	到github.com。然后就可以看到自己的主页啦。
![github-main](/images/posts/github/github-main.png)

2. 在右上角找到“Create a new repo”按钮，创建一	个仓库，仓库名为：`username.github.io`,其	中 `username` 是你的用户名。

3. 在Git Shell下运行下列命令完成本地仓库和远程仓	库的关联。在本地仓库下运行命令：  
	`git remote add origin 	git@github.com:yourmichael/	yourrepertory.git`  
	请千万注意，把上面的yourmichael替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。

	下一步，就可以把本地库的所有内容推送到远程库上：  
	`git push -u origin master`

	推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样。从现在起，只要本地作了提交，就可以通过命令：  
	`git push origin master`

	把本地`master分支`的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！

![github-store](/images/posts/github/github-store.png)

接下来要做的就是生成GitHub Pages页面并发布在`https://username.github.io/`  

首先找到对应仓库的“Settings”
![github-setting](/images/posts/github/github-setting.png)

然后找到Pages页面的自动生成，如果代码符合Jeklly标准，便可以生成初始Blog页面并发布在`https://username.github.io/` 

![Pages-generate](/images/posts/github/Pages-generate.png)


### 利用Jekylly对网页进行渲染

Jekyll 就是一个利用模板化生成HTML的程序，当中的自动化工作都是通过 liquid 实现的。博客搭建剩余的就是常规的前端工作，因此写好 CSS 和 JS 才是王道。

这里在GitHub上有很多正在维护的开源模版可以选择，在模版的基础上可以丰富自己个性化的元素。  

具体的前期jeklly安装和部署可以参看官方文档

<http://jekyll.com.cn/docs>

本地文件的目录结构

![blog-menu](/images/posts/github/blog-menu.png)

各目录的描述和功能如下

![blog-menu-descrip](/images/posts/github/blog-menu-descrip.png)

完成页面的创建后便可以通过Git Shell更新到自己的仓库，Github Pages页面会自动同步更新。接下来的工作就基本是GitHub的操作。

### 利用Markdown书写博客

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。
![Markdown-guide](/images/posts/github/Markdown-guide.png) 

Jekyll 是利用 markdown 书写博客的，所以我们需要熟悉这种语言的基本语法。

参考：<http://wowubuntu.com/markdown/basic.html>

遵守Markdown的基本语法便可以，编辑完成后保存为.md后缀文件便完成了一篇博客的书写，然后通过简单的远程仓库同步，就可以完成本地博客在Pages上的发布。
![Markdown-edit](/images/posts/github/Markdown-edit.png) 


**这不是一篇好的教程，只是对自己搭建博客过程的简单记录。**

**最后，我想说的是：**
**没有十全十美的教程，如同不存在彻头彻尾的绝望。重要****的是保持住一颗捣腾不安的心以及对知识的渴望与 找寻。**

	