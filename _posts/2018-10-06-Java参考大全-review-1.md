### 一维数组

int month_days[] = new int[12]; 

Java的运行系
统会检查以确保所有的数组下标都在正确的范围以内（在这方面，Java与C/C++从根本上不
同，C/C++不提供运行边界检查）

### 多维数组

	int twoD[][] = new int[4][5]; 

当你给多维数组分配内存时，你只需指定第一个（最左边）维数的内存即可。你可以单独地给余下的维数分配内存。例如，下列程序定
义了一个二维数组，它的第二维的大小是不相等的。

	class TwoDAgain {
	public static void main(String args[]) {
	int twoD[][] = new int[4][];
	twoD[0] = new int[1];
	twoD[1] = new int[2];
	twoD[2] = new int[3];
	twoD[3] = new int[4]; 
	}

Java不支持或不允许指针（或者更恰当地说，Java不支持程序员来访问或修改指针）。Java不允许指针，是因为这样做将允许Java applet（小应用程序）突破Java运行环境和主机之间的防火墙（要知道，指针能放在内存的任何地址，即使是Java运行系统之外的地址）。

### 运算符

算术运算符的运算数必须是数字类型。算术运算符不能用在布尔类型上，但是可以用在char类型上，因为实质上在Java中，char类型是int类型的一个子集。

	ratio = denom == 0 ? 0 : num / denom; 

当Java计算这个表达式时，它首先看问号左边的表达式。如果 denom 等于0，那么在问号和冒号之间的表达式被求值，并且该值被作为整个？表达式的值。如果 denom 不等于零，那么在冒号之后的表达式被求值，并且该值被作为整个？表达式的值。然后将整个？表达式的值赋给变量ratio。

### 继承

一个被定义成private的类成员为此类私有，它不能被该类外的所有代码访
问，包括子类。

super( )中的关键概念。当一个子类调用super( )，它调用它的直接超类的构造函数。这样，super( )总是引用调用类直接的超类。这甚至在多层次结构中也是成立的。还有，super( )必须是子类构造函数中的第一个执行语句。

super( )总是引用子类最接近的超类的构造函数。在类层次结构中，如果超类构造函数需要参数，那么不论子类它自己需不需要参数，所有子类必须向上传递这些参数。

### 多线程

	// This program is not synchronized.
	class Callme {
 	void call(String msg) {
 	System.out.print("[" + msg);
 		try {
 			Thread.sleep(1000);
 		} catch(InterruptedException e) {
 			System.out.println("Interrupted");
 		}
 		System.out.println("]");
 	}
	}
	class Caller implements Runnable {
 		String msg;
 		Callme target;
 		Thread t;
 		public Caller(Callme targ, String s) {
 			target = targ;
 			msg = s;
 			t = new Thread(this);
 			t.start(); 
		}
 		public void run() {
 			target.call(msg);
 		}
	}
	class Synch {
 		public static void main(String args[]) {
 			Callme target = new Callme();
 			Caller ob1 = new Caller(target, "Hello");
 			Caller ob2 = new Caller(target, "Synchronized");
 			Caller ob3 = new Caller(target, "World");
 		// wait for threads to end
 			try {
 				ob1.t.join();
 				ob2.t.join();
 				ob3.t.join();
 			} catch(InterruptedException e) {
 				System.out.println("Interrupted");
		 	}
		 }
	}
该程序的输出如下：

	Hello[Synchronized[World]
	]
	] 

在本例中，通过调用sleep( )，call( )方法允许执行转换到另一个线程。该结果是三个消息字符串的混合输出。该程序中，没有阻止三个线程同时调用同一对象的同一方法的方法存在。这是一种竞争，因为三个线程争着完成方法。例题用sleep( )使该影响重复和明显。在大多数情况，竞争是更为复杂和不可预知的，因为你不能确定何时上下文转换会发生。这使程序时而运行正常时而出错。

为达到上例所想达到的目的，必须有权连续的使用call( )。也就是说，在某一时刻，必须限制只有一个线程可以支配它。为此，你只需在call( ) 定义前加上关键字synchronized，如下：

	class Callme {
 		synchronized void call(String msg) {
 		... 

这防止了在一个线程使用call( )时其他线程进入call( )。在synchronized加到call( )前面以
后，程序输出如下：

	[Hello]
	[Synchronized]
	[World] 
记住，一旦线程进入实例的同步方法，没有其他线程可以进入相同实例的同步方法。然而，该实例的其他不同步方法却仍然可以被调用。

	 // synchronize calls to call()
 	public void run() {
 		synchronized(target) { // synchronized block
 			target.call(msg);
		 } 
	}

这里，call( )方法没有被synchronized修饰。而synchronized是在Caller类的run( )方法中声明的。这可以得到上例中同样正确的结果，因为每个线程运行前都等待先前的一个线程结束。