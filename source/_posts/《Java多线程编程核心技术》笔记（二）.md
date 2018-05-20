---
title: 《Java多线程编程核心技术》笔记（二）:对象及变量的并发访问
date: 2018-02-28 14:26:46
tags:
- Java
- 多线程
- 并发
categories: 学习笔记
---
## 0.学习要点
- synchronized对象监视器为Object时的使用。
- synchronized对象监视器为Class时的使用。
- 非线程安全何时出现。
- 关键字volatile的主要作用。
- 关键字volatile与synchronized的区别及使用情况。

---
## 1.线程安全与非线程安全
1. "非[线程安全](https://baike.baidu.com/item/%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8/9747724?fr=aladdin)"会在多个线程对同一个对象中的实例变量进行并发访问时发生，产生的后果就是"[脏读](https://baike.baidu.com/item/%E8%84%8F%E8%AF%BB/152977?fr=aladdin)"，也就是取到的数据实际是被修改过的。
"线程安全"就是获得的实例变量的值是经过同步处理的。
2. "非线程安全"问题存在于"实例变量中"，如果是方法内部的私有变量，则不存在"非线程安全"问题。这是方法内部变量的私有性造成的。
3. 多个线程共同访问同一个对象中的“实例变量"，则有可能出现"非线程安全"。
4. 使用**synchronized关键字**修饰可能会出现非线程安全的方法，使之上锁变为同步方法，多个线程访问同一个对象中的同步方法时一定是线程安全的。

---
## 2.sychronized同步方法
1. synchronized关键字的锁是对象锁，多个对象会产生多个锁:
公共资源类:
		public class Object2_04_2 {
			//synchronized方法锁的是这个对象而非方法
			synchronized public void methodA(){          
				try{
					System.out.println("begin method A threadName="+Thread.currentThread().getName());
					Thread.sleep(2000);
					System.out.println("end!");
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
			synchronized public void methodB(){                   
				//加上synchronized则要等A线程执行完A方法结束后B线程才能执行方法，对象被锁
				try{
					System.out.println("begin method BthreadName="+Thread.currentThread().getName());
					Thread.sleep(2000);
					System.out.println("end!");
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}
线程A:
		public class Thread2_04_2A extends Thread{
			//synchronized方法锁的是这个对象而非方法
			private Object2_04_2 obj;	
			public Thread2_04_2A(Object2_04_2 obj) {
				super();
				this.obj=obj;
			}
			public void run() {
				super.run();
				obj.methodA();
			}
		}
线程B:
		public class Thread2_04_2A extends Thread{
			private Object2_04_2 obj;
			public Thread2_04_2B(Object2_04_2 obj) {
				super();
				this.obj=obj;
			}
			public void run() {
				super.run();
				obj.methodA();
			}
		}
测试类:
		public class Test{
			public static void main(String[] args) {
				Object2_04_2 obj=new Object2_04_2();
				Thread2_04_2A a=new Thread2_04_2A(obj);
				a.setName("a");
				Thread2_04_2B b=new Thread2_04_2B(obj);  //A与B相同
				b.setName("b");
				a.start();
				b.start();
			}
		}
测试结果:
		begin method A threadName=a
		begin method A threadName=a
		end!
		begin method B threadName=b
		end!
		begin method B threadName=b
		end!
结论：
	>1.A线程先持有object对象的lock锁，B线程可以以异步方式调用object对象中的非synchronized类型方法
	>2.A线程先持有object对象的lock锁，B线程如果在这时调用object对象中synchronized类型方法则需要等待，也就是同步。
2. 脏读(dirtyRead):
脏读发生的情况是在读取实例变量时，**此值已经被其他线程更改过了**。
脏读是通过**synchronized关键字**解决的。
3. synchronized锁重入:
关键字synchronized拥有锁重入功能，也就是在使用synchronized时，当一个线程得到一个对象锁后，再次请求此对象锁时是可以再次得到该对象锁的。如:
		public class MyThread extends Thread{
			synchronized public void run（）{
				methodA();
			}
			synchronized public void methodA(){...}
		}
当启动MyThread的对象线程时，调用methodA()方法时锁并未释放，但是可以再次获得该对象锁。不会产生[死锁](1)。
证明在一个synchronized方法/块内部调用本类其他synchronized方法/块时，是永远可以得到锁的。
"[可重入锁](http://blog.csdn.net/johnking123/article/details/50043961)"的概念是：自己可以再次获取自己的内部锁。
可重入锁可以用于父子继承的环境中，子类完全可以通过"可重入锁"调用父类的同步方法。
4. **出现异常，锁会自动释放**：
当一个线程出现异常时，其所持有的锁会自动释放。
5. 同步**不具有继承性**：
		public class AA{
			synchronized public void testMethod(){...}
		}
		public class BB extends AA{
			public void testMethod(){...}
		}
当多个线程访问同一个BB类对象的testMethos()方法时，不会出现同步。若BB未重写testMethod()方法，则仍会出现同步。若一个调父类，一个调子类，则不同步。

---
## 3.synchronized同步代码块
1. synchronized关键字的弊端:
若A线程与B线程访问同一对象的同步方法，若A的同步方法执行了一个时间很长的任务，则B也必须等待很长的时间。
解决方法:使用synchronized(this)同步代码段。
2. synchronized(this)同步代码段的使用:
		public class Task{  //公共资源类
			public void methodA(){
				//一些执行时间长但是不会产生非线程安全的代码段
				synchronized(this){  //使用synchronized(this)同步语句块
					//会出现非线程安全的代码段
				}
			}
			synchronized public void methodB(){  //使用synchronized关键字
				//一些执行时间长但是不会产生非线程安全的代码段
				//会出现非线程安全的代码段
			}
		}
一半同步，一半异步:
不在synchronized(this)与同步方法中的代码异步，在的同步。
3. synchronized(this)锁的仍然是**对象**。
上述代码中,如有两个线程A与B分别同时访问同一个Task类对象的methodA()与methodB()方法时，当A线程通过synchronized(this)获得**对象锁**时，B线程必须要等A线程释放锁才能执行methodB()中的代码，反之亦然。
4. synchronized(任意非this对象)同步代码块:
java支持将"任意对象"作为"对象监视器"来实现同步功能。
synchronized(非this对象)同步代码块的作用:
(1).在多个线程持有的"对象监视器"为同一个的对象的前提下，同一时间只能有一个线程可以执行synchronized(非this对象)同步代码块中的代码。
(2).当持有的"对象监视器"为同一个的对象的前提下，同一时间只能有一个线程可以执行synchronized(非this对象)同步代码块中的代码。
String对象也可以当做对象监视器：
		public class Task{
			public void method(){
				synchronized("哈哈哈"){
					//同步代码块A
				}
				synchronized("哈哈哈"){
					//同步代码块B
				}
			}
		}
则代码块A与代码块B是同步的
synchronized(非this对象)同步代码块不与synchronized(this)对象和synchronized方法争抢锁
5. 两个线程同时执行分支判断时，可能会出现逻辑上的错误，可能会出现脏读。
6. synchronized(非this对象x)的三个重要结论:
(1).多个线程同时执行synchronized(x){}同步代码块时呈同步效果。
(2).当其他线程执行x对象中的synchronized同步方法时呈同步效果。
(3).当其他线程执行x对象中的synchronized(this)同步代码块时呈同步效果。

---
## 4.静态锁，synchronized(class),死锁等
1. 静态synchronized方法
关键字synchronized还可以应用在static方法上，如果这样写，那是对当前的*.java文件对应的Class类进行持锁。如:
		public class test{
			synchronized public static void methodA(){
			//代码段A
			}
			synchronized public void methodB(){
			//代码段B
			}
		}
(1).若有两个线程同时访问test类不同对象的methodA()方法，出现同步互斥。
(2).若有两个线程同时分别访问test类相同对象的methodA()和methodB()方法时，则是异步。因为一个是给Class上锁，一个是给对象上锁，一个线程得到了Class锁，一个线程得到了对象锁。
2. synchronized(Class)同步代码块
作用其实和静态synchronized方法的作用一样。也是锁Class。
		public class Test{
			public static void methodA(){
				synchronized(Test.class){   //代码段A
				}
			}
			public void methodB(){
				synchronized(Test.class){   //代码段B
				}
			}
		}
这时有两个线程同时分别访问Test类不同对象的methodA()和methodB方法时，都会出现同步，因为要获取的都是Test.class锁。
3. synchronized(string)的问题
工作类:
		public class Task {
			// 【String常量池的特性】给synchronized(String)带来的例外
			public static void print(String param){
				try{
					synchronized(param){
						while(true){
							System.out.println(Thread.currentThread().getName());
							Thread.sleep(1000);
						}
					}
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}
两个线程:
		public class Thread2_22_1A extends Thread{
			private Task obj;
			
			public Thread2_22_1A(Task obj) {
				super();
				this.obj=obj;
			}
			@Override
			public void run() {
				super.run();
				obj.print("AAA");
			}
		}
		public class Thread2_22_1B extends Thread{
			private Task obj;
			
			public Thread2_22_1B(Task obj) {
				super();
				this.obj=obj;
			}
			@Override
			public void run() {
				super.run();
				obj.print("AAA");  //与A相同的String传入
			}
		}
测试类：
		public class Test{
			public static void main(String[] args) {
				Task obj=new Task();
				Thread2_22_1A a=new Thread2_22_1A(obj);
				a.setName("A");
				a.start();
				Thread2_22_1B b=new Thread2_22_1B(obj);
				b.setName("B");
				b.start();
			}
		}
运行结果:
		A
		A
		A
		A
		A
		...
说明:
A与B线程持有的是相同的锁("AAA")。
所以一般不用String作为锁对象，而改用其他，如使用new Object()传入一个对象实现**异步**，但是它并不放入缓存中。
4. [死锁](1):
不同的线程都在等待不可能被释放的锁，从而导致所有的任务都无法完成。
**死锁是必须避免的**，这会造成程序的假死。
相互等待对方释放锁就有可能出现死锁。
死锁的判断可以参考博客:[java死锁的排查](http://blog.csdn.net/sidihuo/article/details/52474227)。

---
## 5.内置类与同步
1. 内置类
关于java内部类具体内容可以参考博客:
[java四种内部类详解](http://blog.csdn.net/u011424470/article/details/52319069)
[深入理解Java中为什么内部类可以访问外部类的成员](http://blog.csdn.net/zhangjg_blog/article/details/20000769)
普通内置类的使用:
		public class OutClass{
			class InClass{
			}
		}
		//测试类中实例化
		public class test{
			public static void main(String args[]){
				InClass inclass=new OutClass().new InClass();  //普通内部类的实例化
			}
		}
若OutClass与test不在同一个包内，则需要将InClass内置声明成public的。
2. 静态内置类
		public class OutClass{
			static class InClass{
			}
		}
		//测试类中实例化
		public class test{
			public static void main(String args[]){
				InClass inclass=OutClass.new InClass();  //静态内部类的实例化
			}
		}
3. 内置类的同步:
		public class Run{
			static public class InnerClass1{
				public void method1(InnerClass2 class2){
					synchronized (class2) {
					//代码段
					}
				}
				public synchronized void method2(){
				//代码段
				}
			}
			//静态内部类2
			static public class InnerClass2{
				public synchronized void method3(){
				//代码段
				}
			}
		}
同步代码块synchronized(class2)对class2上锁后，其他线程只能同步调用class2对象中的方法。
4. 锁对象的改变
公共资源类:
		public class Object2_28_1 {
			private String lock="123";
			public void testMethod(){
				try{
					synchronized (lock) {
						//输出
						lock="456";      //锁对象改变
						Thread.sleep(2000);
						//输出
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
两个线程:
		public class Thread2_28_1A extends Thread{
		
			private Object2_28_1 obj;
			
			public Thread2_28_1A(Object2_28_1 obj){
				this.obj=obj;
			}
			public void run(){
				super.run();
				obj.testMethod();
			}
		}
		public class Thread2_28_1B extends Thread{
		
			private Object2_28_1 obj;
			
			public Thread2_28_1B(Object2_28_1 obj){
				this.obj=obj;
			}
			public void run(){
				super.run();
				obj.testMethod();
			}
		}
测试类:
		public class Test2_28_1 {
			public static void main(String[] args) throws InterruptedException {
				Object2_28_1 obj=new Object2_28_1();
				Thread2_28_1A a=new Thread2_28_1A(obj);
				a.setName("aaa");
				Thread2_28_1B b=new Thread2_28_1B(obj);
				b.setName("bbb");
				a.start();
				Thread.sleep(50);           //不加这句同步
				b.start();
			}
		}
结果:
(1).若加上Thread.sleep(50)，则异步，因为50毫秒后线程B获得的锁是"456"。
(2).若不加Thread.sleep(50)，则同步，即使对象改变，线程B从一开始等待"123"锁，不会改变等待的锁。
还有一个结论:
只要作为对象监视器的对象，即使对象的属性改变了，运行结果也还是同步的。

---
## 6.volatile关键字
1. Thread与Runnable
不是在多继承的情况下使用继承Thread和实现Runnable取得的实验结果并没有什么太大区别。但是一旦出现多继承，就得用实现Runnable接口的方式。
2. volatile关键字的用法:
**强制从公共堆栈中读取变量的值**。
不使用volatile出现死循环的情况:
		public class Thread2_31 extends Thread{
			//volatile解决异步死循环
			
			private boolean isRunning=true;                  //在_server运行环境下死循环
			//volatile private boolean isRunning=true;
		
			public boolean isRunning() {
				return isRunning;
			}
			public void setRunning(boolean isRunning) {
				this.isRunning = isRunning;
			}
			@Override
			public void run() {
				super.run();
				System.out.println("进入run了!");
				while(isRunning){
					System.out.println("123");
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				System.out.println("线程被停止了!");
			}
		}
		
		public class Test2_31 {
			public static void main(String[] args) {
				try{
					Thread2_31 thread=new Thread2_31();
					thread.start();
					Thread.sleep(1000);             //thread线程停止1秒
					thread.setRunning(false);
					System.out.println("已经赋值为false");
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}
以上代码在JVM设置为-server时会出现死循环。原因是-server为了运行效率，一只获取的是私有堆栈中的isRunning值，而`thread.setRunning(false);`更新的是公有堆栈中的值。所以程序会一直死循环。
关于JVM-server的具体内容可以参考博客:[关于JVM的类型和模式](http://www.importnew.com/20715.html)。
3. **volatile与synchronized的区别**:
(1).volatile是synchronized的轻量级实现，性能肯定比synchronized好，但是volatile只能修饰变量，synchronized可以修饰方法以及代码块。新版本的JDK发布使得synchronized的性能有了很大提升。
(2).多线程访问volatile不会阻塞，访问synchronized会发生阻塞。
(3).volatile能保证数据的可见性，**不能保证原子性**;synchronized既可以保证原子性，也可以间接保证可见性，他会将私有内存和公共内存中的数据同步化。
(4).关键字volatile保证了数据的可见性;而synchronized保证了多个线程访问同步资源的可见性。
4. volatile的非原子性(致命缺陷):
对于volatile修饰的变量，JVM只是保证从内存加载到线程工作内存的值是最新的，但是多个线程同时访问同一个实例变量并修改时，无法保证同步修改(非原子操作时)，所以在多个线程访问同一个实例变量还是需要加锁。
		public class Thread2_32_1 extends Thread{
			//volatile的非原子特性
			volatile public static int count;
			private static void addCount(){
				for(int i=0;i<100;i++){
					count++;	//这一步是非线程安全的
				}
				System.out.println("count="+count);
			}
			
			public void run() {
				// TODO Auto-generated method stub
				addCount();
			}
		}
具体的解释:
变量在内存中的工作阶段:
(1).read和load阶段:从主存复制变量到当前线程工作内存
(2).use和assign阶段:执行代码阶段，改变变量值
(3).store和write阶段:用对应数据刷新主存对应变量的值
多线程环境中use和assign是多次出现的，load,use,assign都是非线程安全的
5. synchronized代码块具有volatile同步的功能
		public class Object2_35 {
			//synchronized代码块有volatile同步功能          
			private boolean isRun=true;
			public void runMethod(){
				while(isRun){
					synchronized ("AAA") {               //可视,_server服务器环境下
					}
				}
				System.out.println("停了");
			}
			public void stopMethod(){
				isRun=false;
			}
		}
同步synchronized不仅可以解决一个线程看到对象处于不一致的状态，还可以保证进入同步方法/代码块的每个线程，都看到由一个锁保护之前的所有修改结果。

---
## 7.原子类(Atomic)
1. 使用[原子类](http://blog.csdn.net/youyou1543724847/article/details/52735510)进行i++操作:
除了在i++时使用synchronized关键字外，还可以使用AtomicInteger原子整型来保证同步。
2. 原子操作是不可分割的整体，没有其他线程能够中断或者检查正在原子操作中的变量，一个原子类型就是一个原子操作的可用类型，可以在没有锁的情况下做到线程安全。
		public class Thread2_33 extends Thread{
			//使用原子类对i++进行操作
			
			private AtomicInteger count=new AtomicInteger(0);
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				super.run();
				for(int i=0;i<10000;i++){
					System.out.println(count.incrementAndGet());      //count++操作
				}
			}
		}
3. 原子类也并非绝对线程安全的:
原子操作在具有逻辑性的情况下输出结果也具有随机性。
		public class Object2_34 {
			//有原子类也并非完全安全
			
			public static AtomicLong along=new AtomicLong();
			
		//	synchronized public void addNum(){                 //需要同步,两方法之间不是原子的
			public void addNum(){
				System.out.println(Thread.currentThread().getName()+" 加了100后的值是："+along.addAndGet(100));
				along.addAndGet(1);
			}
		}
		
		public class Test2_34 {
			
			public static void main(String[] args) {
				try{
					Object2_34 obj=new Object2_34();
					Thread2_34[] array=new Thread2_34[5];
					for(int i=0;i<array.length;i++){
						array[i]=new Thread2_34(obj);
					}
					for(int i=0;i<array.length;i++){
						array[i].start();
					}
					Thread.sleep(1000);
					System.out.println(obj.along.get());
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}
方法内部可能是原子的，但是方法之间是非线程安全的，解决方法仍是synchronized关键字。
4. 更多关于java原子类的知识可以访问博客:[Java并发:原子类](http://blog.csdn.net/youyou1543724847/article/details/52735510)。
---
## 8.总结：
学习多线程并发，要着重**"外练互斥，内修可见"**。

---
[1]:(https://baike.baidu.com/item/%E6%AD%BB%E9%94%81/2196938?fr=aladdin)