---
layout: post
title: Python问题总结（持续更）
categories: Python
description: python的学习
keywords: python,basic
---

##### 递归中的栈溢出问题

计算阶乘的函数最简单的实现方法就是递归。  
可以通过以下函数定义实现：
  
     def fact(n):
    	if n==1:
        	return 1
   		return n * fact(n - 1)  

我们可以通过调用`fact(5)`计算出5！的值，这不会有任何问题。但试着计算`fact(1000)`时，就会出现问题了。  

    >>> fact(1000)
	Traceback (most recent call last):
  	File "<stdin>", line 1, in <module>
  	File "<stdin>", line 4, in fact
  	...
  	File "<stdin>", line 4, in fact
	RuntimeError: maximum recursion depth exceeded  

出现这种情况是因为出现了栈溢出。在计算机中，函数调用是通过栈调用实现的，每进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。  

解决这个问题的地方是通过尾递归优化。事实上尾递归和循环的效果是一样的，所以，把循环看成是一种特殊的尾递归函数也是可以的。

尾递归是指，在函数返回的时候，调用自身本身，并且，return语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。

上面的fact(n)函数由于return n * fact(n - 1)引入了乘法表达式，所以就不是尾递归了。要改成尾递归方式，需要多一点代码，主要是要把每一步的乘积传入到递归函数中：

	def fact(n):
    return fact_iter(n, 1)

	def fact_iter(num, product):
    	if num == 1:
        	return product
    	return fact_iter(num - 1, num * product)

##### 默认参数的坑
看下面定义的函数：

	def add_end(L=[]):
    L.append('END')
    return L

当你正常调用时，结果很Ok：

	>>> add_end([1, 2, 3])
	[1, 2, 3, 'END']
	>>> add_end(['x', 'y', 'z'])
	['x', 'y', 'z', 'END']

当使用默认参数调用时：

	>>> add_end()
	['END']

当我们再次使用默认参数调用时，结果和我们所想的不一样：

	>>> add_end()
	['END', 'END']

原因解释如下：

Python函数在定义的时候，默认参数L的值就被计算出来了，即[]，因为默认参数L也是一个变量，它指向对象[]，每次调用该函数，如果改变了L的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的[]了。

`所以，定义默认参数要牢记一点：默认参数必须指向不变对象！`

知道原因后，可以对上面的代码进行修改如下：

	def add_end(L=None):
	if L is None:
		L=[]
	L.append('END')
	return L

这时，我们无论调用多少次，都不会有问题了！

	>>> add_end()
	['END']
	>>> add_end()
	['END']

为什么要设计str、None这样的不变对象呢？因为不变对象一旦创建，对象内部的数据就不能修改，这  样就减少了由于修改数据导致的错误。此外，由于对象不变，多任务环境下同时读取对象不需要加锁，同时读一点问题都没有。我们在编写程序时，如果可以设计一个不变对象，那就尽量设计成不变对象。




	



