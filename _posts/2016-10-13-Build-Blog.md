---
layout: post
title: 使用Github和Jekyll搭建个人博客
categories: Github
description: 一直都有搭建一个个人博客的想法，刚好近日空闲时间比较多，于是用了一天的时间完成了第一版的个人独立博客的搭建
keywords: blog，git，jeklly
---


搭建个人博客的想法已经出现过很多次，因为养成写博客的习惯，不仅可以让我们不断总结过去的知识，还可以增强我们做事的条理性和逻辑性，但由于各种事情和自己拖延症的毛病，迟迟没有进行。今天终于用了一天的时间基本完成了独立博客的搭建。

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
	