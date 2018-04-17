---
title: 《Java多线程编程核心技术》笔记（一）:java多线程基本操作
date: 2018/2/27 19:19:40  
tags:
 - Java
 - 多线程
categories: 学习笔记

---
## 0.学习要点
- 线程的启动
- 如何使线程暂停
- 如何使线程停止
- 线程的优先级
- 线程安全相关的问题

---
## 1.进程与线程概念及多线程优点
1. 进程：
	>是一次程序的执行；是一个程序及其数据在处理机上顺序执行时所发生的活动；是程序在一个数据集合上运行的过程；是系统进行资源分配和调度的一个独立单位
2. 线程：
	>在进程中独立运行的子任务
3. 多线程是异步的，而且线程被调用的时机是随机的。代码顺序≠程序执行顺序。

---
## 2.如何使用多线程
1. 关于main线程：
在public void static main()方法中打印:
		System.out.println(Thread.currentThread().getName());
控制台输出的结果为`main`,说明这时有一个main线程在执行，由JVM创建。但是这个main和main方法没有任何关系，仅仅只是名字相同而已。
2. 继承Thread类：
Thread类的结构:
		public Thread implements Runnable
使用Thread和使用Runnable没有本质区别。只是使用Thread不能多继承。
使用Thread：
		public class MyThread extends Thread {
			@override
			public void run(){...}
		}
		public class run{ public static void main(String args[]){
			MyThread mythread =new MyThread();
			mythread.start();
		}
一定要重写run()方法。
3. 实现Runnable接口:
先来看看Thread的构造方法:
		public Thread( );
		public Thread(Runnable target);
		public Thread(String name);
		public Thread(Runnable target, String name);
		public Thread(ThreadGroup group, Runnable target);
		public Thread(ThreadGroup group, String name);
		public Thread(ThreadGroup group, Runnable target, String name);
		public Thread(ThreadGroup group, Runnable target, String name, long stackSize);
详情请参考博客[Thread 的构造方法](http://blog.csdn.net/JackieLiuLixi/article/details/37569575)。
第二个和第四个构造方法说明Thread通过传入一个Runnable对象而产生一个Thread对象。那么使用Runnable接口的方法如下:
		public class runnable implements Runnable{
			@override
			public void run(){...}
		}
		public class Test1_4 {
			//线程的另一种实现方式：实现runnable接口
			//Thread类也是通过实现该接口
			public static void main(String[] args) {
				runnable run=new runnable();
				Thread thread=new Thread(run，A); //线程thread的name设为A，可通过getName()方法得到
				thread.start();
				System.out.println("main线程结束！");
			}
		}
因为Thread实现了Runnable接口，所以也可以传入Thread类对象。
4. 随机性:
多线程具有随机的特性，**执行start()的顺序不代表线程启动的顺序**。

---
## 3.关于非线程安全
1. 非线程安全:
	>指多个线程对同一个对象中的同一个实例变量进行操作时会出现值被更改、值不相同的情况。进而影响程序执行流程。
2. 如何解决非线程安全:（简要方法）
使用在可能出现非线程安全的方法前使用**synchronized**关键字，如：
		public class MyThread extends Thread{
			@override
			synchronized public void run(){...}
		}
3. i--,i++与System.out.println()联用可能会产生的异常:
在线程中使用如下代码:
		System.out.println("i=" + (i++) + "threadName=" + Thread.currentThread.getName());
时有发生非线程安全的概率，因为println()虽然是线程安全的，但是i++是在进入println()之前发生的。解决方法依然是使用同步。

---
## 4.一些Thread常用方法
1. currentThread()方法:
*currentThread()方法可以返回代码段正在被哪个线程调用的信息。*
this则返回当前所处的线程，this.getName()得到线程名。
有一个线程:
		public class MyThread extends Thread{
			public MyThread(){
				System.out.println(Thread.currentThread.getName());
				System.out.println(this.getName());
			}
			public void run{
				System.out.println(Thread.currentThread.getName());
				System.out.println(this.getName());
			}
		}
测试:
		public static void main(String[] args) {
			MyThread thread=new MyThread();    
			Thread t=new Thread(thread);           //将thread交给t执行
			t.setName("A");
			t.start();
			thread.run();
		}
结果：
		main
		Thread-0
		A
		Thread-0
		main
		Thread-0
这里输出的Thread-0实际上是thread的name。
构造函数被main线程调用，run()方法是被线程A自动调用的，直接调用run()方法时Thread.curretThread时main线程。
具体this.getName()与Thread.currentThread.getName()区别可以参考博客：[多线程当中this.name和Thread.currentThread.getName的区别](http://blog.csdn.net/qq_34970891/article/details/78881463)。
2. isAlive()方法:
*判断当前线程是否处于活动状态。
活动状态：线程处于正在运行或准备开始运行的状态。*
判断当前线程是否存活:
		this.isAlive();
判断上级线程是否存活：
		Thread.currentThread().isAlive();
判断某一线程是否存活：
		Thread t=new Thread();
		t.isAlive();
3. sleep()方法：
*在指定毫秒内让当前“正在执行的线程”休眠（暂停执行）。
这个正在执行的线程>是指`this.currentThread()`返回的线程。*
this.currentThread().getName()=this.getName()
(我的理解，如有异议请告知）
注意:sleep()是静态方法，调用方式为：
		Thread.sleep();  //括号内为毫秒数
4. getId()方法:
*取得线程的唯一标识符*

---
## 5.线程的中断与判断
1. 线程中断（与停止区分开，中断只是一种状态，不会真正停止线程）：
thread.interrupt()方法（非静态）:
**中断一个线程，在线程thread中打了一个中断标记，并不会真正停止线程**。
2. Thread.interrupted()方法:
*静态方法，判断当前线程是否中断，当前线程是指执行Thread.interrupted()方法的线程，**执行后清除当前线程的中断标记**。*
3. thread.isInterrupted()方法:
*非静态方法，判断一个线程(thread)是否是中断状态，**执行后不清除该线程的中断标记***。

---
## 6.停止线程的几种方法
1. 异常法(推荐):
		public class Thread1_18_2 extends Thread{
			//for语句加break实现部分中断，for语句下的语句还会继续行
			//使用throw new InterruptedException()抛出异常则不会
			public void run(){
				super.run();
				try{
					for(int i=0;i<200;i++){
						System.out.println("i="+i);
						if(this.interrupted()){
							throw new InterruptedException("线终止啦");
						}
					}
					System.out.println("这是for语句后面的语句，是执行");
				}catch(InterruptedException e){
					System.out.println("成功进入threadcatch");
					e.printStackTrace();
				}
			}
		}
测试代码:
		public static void main(String[] args) {
			try{
				Thread1_18_2 thread=new Thread1_18_2(); 
				thread.start();
				Thread.sleep(10);
				thread.interrupt();
			}catch(InterruptedException e){
				System.out.println("main catch");
				e.printStackTrace();
			}
			System.out.println("end!");
		}
2. 沉睡异常法（Thread.sleep()）(推荐):
先中断再沉睡:
		public class Thread1_19_2 extends Thread{
			//在interrput()状态下使用sleep()的情况
			//抛出异常从而停止线程（推荐使用）
			public void run(){
				super.run();
				try{
					for(int i=0;i<1000;i++){
						System.out.println("i="+(i+1));
					}
					System.out.println("run begin");
					Thread.sleep(1000);
					System.out.println("run end");
				}catch(InterruptedException e){
					System.out.println("先停止，再遇见sleep!出错进入catch");
					e.printStackTrace();
				}
			}
			public static void main(String[] args) {
				Thread1_19_2 thread =new Thread1_19_2();
				thread.start();
				thread.interrupt();
				System.out.println("end!");
			}
		}
运行结果:
		run begin
		先停止，再遇见sleep!出错进入catch
		java.lang.InterruptedException: sleep interrupted
			at java.lang.Thread.sleep(Native Method)
			at Chapter1.Thread.Thread1_19_2.run(Thread1_19_2.java:14)
若调换main()方法中的thread.start()与thread.interrupt()的位置，则不会抛异常，thread线程也不会停止;
先sleep()再中断也可以停止线程，效果与上述相同。
3. 暴力法（不推荐使用）:
使用stop()方法:
	>1.会抛出java.lang.ThreadDeath异常，不用显式捕捉
	>2.会使一些清理性工作得不到完成
	>3.会对锁定的对象解锁，导致数据得不到同步处理，出现数据不一致的结果(使sychornized失效)
4. return法停止线程:
		public class Thread1_21 extends Thread{
			//interrupt()与return结合使用实现停止线程（可以使用）
			public void run(){
				while(true){
					if(this.isInterrupted()){
						System.out.println("线程停止了");
						return;
					}
					System.out.println("timer="+System.currentTimeMillis());
				}
			}
			public static void main(String[] args) throws InterruptedException {
				Thread1_21 thread=new Thread1_21();
				thread.start();
				Thread.sleep(2000);
				thread.interrupt();
			}
		}
不过还是建议使用异常法，因为在catch中可以将异常向上拋，使线程的停止事件得以传播。

---
## 7.线程的挂起/暂停与恢复
1. suspend()与resume()方法:
		Thread thread =new Thread();
		thread.start();
		thread.suspend();
		thread.resume();
但是会看到~~suspend()~~和~~resume()~~,说明它们已经不被推荐使用。
2. suspend()与resume()方法的缺点:
	>独占：
	>>使用不当会造成公共同步对象的独占，使其它线程无法访问公共资源。
	>
	>不同步:
	>>容易出现因线程暂停而导致数据不同步的现象。
3. yield()方法:
*yield()方法的作用是放弃当前的cpu资源，将它让给其他任务去占用CPU执行时间。*
但是放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片。
测试代码:
		public class Thread1_25 extends Thread{
			//yield()方法：使线程放弃当前CPU资源，将它让给其他任务去占用cpu执行时间		
			public void run(){
				long begin=System.currentTimeMillis();
				int count=0;
				for(int i=0;i<500000;i++){ 
					Thread.yield();            //加上注释可以测试yield是否让资源
					count=count+1;
				}
				long end=System.currentTimeMillis();
				System.out.println("用时："+(end-begin)+"毫秒!");
			}
			public static void main(String[] args) {
				Thread1_25 thread=new Thread1_25();
				thread.start();
			}
		}

---
## 8.线程的优先级
1. 简介:
线程可以划分优先级，优先级较高的线程得到的CPU资源较多，也就是CPU优先执行优先级较高的线程中的任务。
设置线程优先级:
		thread.setPriority()；
获取线程优先级:
		thread.getPriority();
setPriority()方法的实现：
	    public final void setPriority(int newPriority) {
	        ThreadGroup g;
	        checkAccess();
	        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
	            throw new IllegalArgumentException();
	        }
	        if((g = getThreadGroup()) != null) {
	            if (newPriority > g.getMaxPriority()) {
	                newPriority = g.getMaxPriority();
	            }
	            setPriority0(priority = newPriority);
	        }
	    }
java中线程的优先级有10个等级（1~10）,如果小于1或者大于10则JDK会抛出异常throw new IllegalArgumentException()（非法参数异常）。
java有三个优先级常量:
	    public final static int MIN_PRIORITY = 1;
		
	    public final static int NORM_PRIORITY = 5;  //默认值
		
	    public final static int MAX_PRIORITY = 10;
2. 优先级的继承性:
**A线程启动B线程，则B线程优先级与A相同**。
		public class Thread1_26_1 extends Thread{
			//线程1
			public void run(){
				System.out.println("thread1 的 priority="+this.getPriority());
				Thread1_26_2 thread2=new Thread1_26_2();  //1线程启动2线程
				thread2.start();
			}
		}
		public class Thread1_26_2 extends Thread{
			//线程2
			public void run(){
				System.out.println("thread2的priority="+this.getPriority());
			}
		}
		public class test{
			public static void main(String[] args) {
				System.out.println("main 的 priority="+Thread.currentThread().getPriority());
				Thread.currentThread().setPriority(6);    //注释这一行测试
				System.out.println("main 的 priority="+Thread.currentThread().getPriority());
				Thread1_26_1 thread=new Thread1_26_1();  //main线程启动线程1
				thread.start();
			}
		}
测试结果1（未给main线程重新设置优先级）:
		main 的 priority=5
		main 的 priority=5
		thread1 的 priority=5
		thread2的priority=5
测试结果2（将main线程优先级重设为6）:
		main 的 priority=5
		main 的 priority=6
		thread1 的 priority=6
		thread2的priority=6
3. 优先级的规则性:
规则:**CPU尽量将执行资源让给优先级比较高的线程**。
当现场优先级等级差距很大时，谁先执行完代码和代码的调用顺序无关。
4. 优先级的随机性:
**优先级较高的线程不一定每一次都先执行完**。
不要把线程的优先级与运行结果的顺序作为衡量的标准。（如优先级为5和优先级为6的线程）
5. 优先级高的线程运行得快。

---
## 9.守护线程(后台线程)
1. java中有两种线程，一种是用户线程，另一种是守护线程
守护线程**是一种特殊的线程，当进程中不存在非守护线程时，守护线程自动销毁**，典型的守护线程就是JVM的垃圾回收(GC)线程了。
2. 设置守护线程:
Thread thread =new Thread();
thread.setDaemon(true);  //s=设为守护线程,需在start()前
thread.start();
3. 更多关于守护线程的知识可以查看博客:[守护线程与非守护线程](https://www.cnblogs.com/lixuan1998/p/6937986.html)。

---
