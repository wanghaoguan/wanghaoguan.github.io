---
layout: post
title:  Java基础笔记（一）
categories: Java
description: Java的学习
keywords: Java,basic
---
##### 栈和堆

在java中，我们关心内存中的两个区域：对象的生存空间堆（heap）和方法调用及变量的生存空间栈（stack）   
所有创建的对象，不论实际类型，都会是Object的一个实例，在堆上除了会有一个创建的对象外，还包括Object对象中的所有内容。  

	当你把对象装进ArrayList<Object>时，不管它原来是什么，都只能把它当作Object对象。  
	从ArrayList<Object>中取出引用时，取出的引用类型也只能是Object。 


 
正在执行的方法会放在栈顶，被调用方法会放在调用方法之上 
##### 
* 局部变量是声明在方法内或方法的参数上；局部变量生存在栈上
* 实例变量是声明在类中方法之外的地方；实例变量存在于对象所属的堆上

##### 设计类、子类、抽象类、接口
* 如果新的类无法对其他的类通过IS-A测试时，就设计不继承其他类的类 
* 只有在需要某类的特殊化版本时，以覆盖或增加新的方法来继承现有的类
* 当你需要定义一群子类的模板，又不想让程序员初始化此模板时，就需要设计出抽象的类
* 如果想要定义出类可以扮演的角色，使用接口


##### SUMMARY
* 如果不想让某个类被初始化，就以abstract关键词将它标记为抽象
* 抽象的类可以带有抽象和非抽象的方法；带有抽象方法的类必定是抽象类
* 抽象的方法没有内容，声明以分号结束
* 抽象的方法必须在具体的类中运行
* 不管实际上所引用的对象是什么类型，只有在引用变量的类型就是带有某方法的类型时才能调用该方法（？_？）
* Object引用变量在没有类型转换的情况下不能赋值给其他的类型，若堆上的类型与所要转换的类型不兼容，此转换会在执行期产生异常

	eg:  Dog d = (Dog) x.getObject(aDog);

* 从ArrayList<Object>取出的对象只能被Object引用，其他都需要用类型转换来改变
* 多重继承会导致致命方块的问题
* 接口是100%纯天然抽象类‘使用interface关键词取代class来声明接口；实现接口时使用implement关键词；一个类可以实现多个接口
* 实现某接口的类必须实现它所有的方法
* 从子类调用父类的方法使用super关键词

##### 构造函数
* 构造函数是新建类时会执行的程序  
 `Duck d = new Duck();`
* 构造函数与类的名字一样，且没有返回类型  
 `public Duck(int size){}`
* 如果没有写构造函数，编译器会默认一个没有参数的构造函数  
 `public Duck(){}`
* 一个类可以有多个构造函数，但参数类型或顺序必须有不同
* 在创建新对象时，所有继承下来的构造函数都会执行
* 抽象的类也有构造函数。因为虽然不能对抽象的类进行new操作，但抽象的类还是父类，它的构造函数会在具体子类创建实例时执行
* 构造函数在执行时会先去执行父类的构造函数，父类的构造函数必须在子类的构造函数之前结束
* 想要调用父类的构造函数，唯一的方法就是通过super()进行调用  

	
		public class Duck extends Animal {  
			int size;
		
			public Duck(int newSize){
				super();
				size = newSize;
			}
		}

	如果自己没有主动调用super(),编译器会帮我们加上super()的调用
* 对super()的调用必须是构造函数的第一个语句
* 使用this()可以从某个构造函数调用同一个类的另外一个构造函数；this()只能用在构造函数中，且必须是第一行语句；super()与this()不能同时调用
* 对象的生命周期要看引用变量的生命周期而定，而引用变量的生命周期要看它是局部变量还是实例变量来确定
* 静态的方法不能调用非静态的变量
* 静态变量的值只能初始化一次，且对所有的实例都相同，同一个类所有的实例变量共享一份静态变量
* 静态变量会在该类的任何对象创建之前和任何静态方法执行之前完成初始化

