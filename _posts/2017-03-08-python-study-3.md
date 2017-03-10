---
layout: post
title: Python中的正则表达式手册
categories: Python
description: python的学习
keywords: python,basic
---

在准备用Python做一个简单的网页爬虫时，遇到了写正则表达式的麻烦，所以这里先整理了python中正则表达式的一些内容。

**以下内容参考于<http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html>**
### 一、正则表达式基础

正则表达式是用于处理字符串的强大工具，它并不是Python的一部分。  
其他编程语言中也有正则表达式的概念，区别只在于不同的编程语言实现支持的语法数量不同。
它拥有自己独特的语法以及一个独立的处理引擎，在提供了正则表达式的语言里，正则表达式的语法都是一样的。  
正则表达式的匹配过程是：  
依次拿出表达式和文本中的字符比较，如果每一个字符都能匹配，则匹配成功；一旦有匹配不成功的字符则匹配失败。如果表达式中有量词或边界，这个过程会稍微有一些不同。  
下面是Python支持的正则表达式元字符和语法：  
![RegularExpression-1](/images/posts/python/RegularExpression.png)
  
**数量词的贪婪模式与非贪婪模式**  
正则表达式通常用于在文本中查找匹配的字符串。Python里数量词默认是贪婪的（在少数语言里也可能是默认非贪婪），总是尝试匹配尽可能多的字符；非贪婪的则相反，总是尝试匹配尽可能少的字符。例如：正则表达式`ab*`如果用于查找`abbbc`，将找到`abbb`。而如果使用非贪婪的数量词`ab*?`，将找到`a`。  

**反斜杠的困扰**  
与大多数编程语言相同，正则表达式里使用`\`作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符`"\"`，那么使用编程语言表示的正则表达式里将需要4个反斜杠`"\\\\"`：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。Python里的原生字符串很好地解决了这个问题，这个例子中的正则表达式可以使用`r"\\"`表示。同样，匹配一个数字的`"\\d"`可以写成`r"\d"`。有了原生字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观。  

### 二、re模块中常用功能函数

##### 1、compile()
编译正则表达式模式，返回一个对象的模式。（可以把那些常用的正则表达式编译成正则表达式对象，这样可以提高一点效率。）

格式：

re.compile(pattern,flags=0)

pattern: 编译时用的表达式字符串。

flags 编译标志位，用于修改正则表达式的匹配方式，如：是否区分大小写，多行匹配等。常用的flags有：  
![compile()-1](/images/posts/python/compile().png)

	import re
	tt = "Tina is a good girl, she is cool, clever, and so on..."
	rr = re.compile(r'\w*oo\w*')
	print(rr.findall(tt))   #查找所有包含'oo'的单词
	执行结果如下：
	['good', 'cool']

##### 2、match()

决定RE是否在字符串刚开始的位置匹配。//注：这个方法并不是完全匹配。当pattern结束时若string还有剩余字符，仍然视为成功。想要完全匹配，可以在表达式末尾加上边界匹配符'$'

格式：

re.match(pattern, string, flags=0)

	print(re.match('com','comwww.runcomoob').group())
	print(re.match('com','Comwww.runcomoob',re.I).group())
	执行结果如下：
	com
	com
##### 3、search()

 格式：

re.search(pattern, string, flags=0)

re.search函数会在字符串内查找模式匹配,只要找到第一个匹配然后返回，如果字符串没有匹配，则返回None。

	print(re.search('\dcom','www.4comrunoob.5com').group())
	执行结果如下：
	4com

注：match和search一旦匹配成功，就是一个match object对象，而match object对象有以下方法：

* group() 返回被 RE 匹配的字符串
* start() 返回匹配开始的位置
* end() 返回匹配结束的位置
* span() 返回一个元组包含匹配 (开始,结束) 的位置
* group() 返回re整体匹配的字符串，可以一次输入多个组号，对应组号匹配的字符串。  

a. group（）返回re整体匹配的字符串，
b. group (n,m) 返回组号为n，m所匹配的字符串，如果组号不存在，则返回indexError异常
c.groups（）groups() 方法返回一个包含正则表达式中所有小组字符串的元组，从 1 到所含的小组号，通常groups()不需要参数，返回一个元组，元组中的元就是正则表达式中定义的组。 

	import re
	a = "123abc456"
 	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(0))   #123abc456,返回整体
 	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(1))   #123
 	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(2))   #abc
 	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(3))   #456
	###group(1) 列出第一个括号匹配部分，group(2) 列出第二个括号匹配部分，group(3) 列出第三个括号匹配部分。###

##### 4、findall()

re.findall遍历匹配，可以获取字符串中所有匹配的字符串，返回一个列表。

 格式：

re.findall(pattern, string, flags=0)

	import re
	tt = "Tina is a good girl, she is cool, clever, and so on..."
	rr = re.compile(r'\w*oo\w*')
	print(rr.findall(tt))
	print(re.findall(r'(\w)*oo(\w)',tt))#()表示子表达式 
	执行结果如下：
	['good', 'cool']
	[('g', 'd'), ('c', 'l')]

##### 5、finditer()

搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器。找到 RE 匹配的所有子串，并把它们作为一个迭代器返回。

格式：

re.finditer(pattern, string, flags=0)

	iter = re.finditer(r'\d+','12 drumm44ers drumming, 11 ... 10 ...')
	for i in iter:
    print(i)
    print(i.group())
    print(i.span())
	执行结果如下：
	<_sre.SRE_Match object; span=(0, 2), match='12'>
	12
	(0, 2)
	<_sre.SRE_Match object; span=(8, 10), match='44'>
	44
	(8, 10)
	<_sre.SRE_Match object; span=(24, 26), match='11'>
	11
	(24, 26)
	<_sre.SRE_Match object; span=(31, 33), match='10'>
	10
	(31, 33)

##### 6、split()

按照能够匹配的子串将string分割后返回列表。

可以使用re.split来分割字符串，如：re.split(r'\s+', text)；将字符串按空格分割成一个单词列表。

格式：

re.split(pattern, string[, maxsplit])

maxsplit用于指定最大分割次数，不指定将全部分割。

	print(re.split('\d+','one1two2three3four4five5'))
	执行结果如下：
	['one', 'two', 'three', 'four', 'five', '']

##### 7、sub()

使用re替换string中每一个匹配的子串后返回替换后的字符串。

格式：

re.sub(pattern, repl, string, count)

	import re
	text = "JGood is a handsome boy, he is cool, clever, and so on..."
	print(re.sub(r'\s+', '-', text))
	执行结果如下：
	JGood-is-a-handsome-boy,-he-is-cool,-clever,-and-so-on...

	其中第二个函数是替换后的字符串；本例中为'-'

	第四个参数指替换个数。默认为0，表示每个匹配项都替换。

re.sub还允许使用函数对匹配项的替换进行复杂的处理。

如：re.sub(r'\s', lambda m: '[' + m.group(0) + ']', text, 0)；将字符串中的空格' '替换为'[ ]'。

	import re
	text = "JGood is a handsome boy, he is cool, clever, and so on..."
	print(re.sub(r'\s+', lambda m:'['+m.group(0)+']', text,0))
	执行结果如下：
	JGood[ ]is[ ]a[ ]handsome[ ]boy,[ ]he[ ]is[ ]cool,[ ]clever,[ ]and[ ]so[ ]on...

##### 8、subn()

返回替换次数

格式：

subn(pattern, repl, string, count=0, flags=0)

	print(re.subn('[1-2]','A','123456abcdef'))
	print(re.sub("g.t","have",'I get A,  I got B ,I gut C'))
	print(re.subn("g.t","have",'I get A,  I got B ,I gut C'))
	执行结果如下：
	('AA3456abcdef', 2)
	I have A,  I have B ,I have C
	('I have A,  I have B ,I have C', 3)

注意：re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配。

	a=re.search('[\d]',"abc33").group()
	print(a)
	p=re.match('[\d]',"abc33")
	print(p)
	b=re.findall('[\d]',"abc33")
	print(b)
	执行结果：
	3
	None
	['3', '3']