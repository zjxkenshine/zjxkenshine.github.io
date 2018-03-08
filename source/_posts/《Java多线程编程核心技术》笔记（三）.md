---
title: 《Java多线程编程核心技术》笔记（三）：线程间的通信
date: 2018-03-01 15:43:51
tags:
- Java
- 多线程
categories: 学习笔记

---
## 0.学习要点
- 使用wait/notify实现线程通信
- 生产者/消费者模式的实现
- 方法join的使用
- ThreadLocal类的使用

---
## 1.等待/通知机制简介
1. 不使用[等待/通知机制](http://blog.csdn.net/fjse51/article/details/54618538)实现线程间通信
线程与线程之间不是独立的个体，它们彼此之间可以互相**通信和协作**。
可以通过sleep()结合while(true)死循环法来实现多个线程通信。
但是使用死循环轮询会浪费CPU资源，所以急需一种机制减少CPU的浪费——**等待/通知机制**。
2. 等待/通知机制方法介绍(wait()/notify()方法):
**wait()方法**:*Object()类对象，使当前线程在wait()代码处停止执行，直到接到通知或被中断为止。wait()是对象级别锁，只能在同步方法或者同步语句块中调用wait()方法。*
**notify()方法**:*也要在同步方法或者同步语句块中调用，即调用前线程必须获得该对象的对象锁。调用notify后不会立刻唤醒等待的线程，而是等notify后的该同步方法或同步语句块中的代码执行完毕释放锁后，才从等待的线程中挑出一个执行。若还现场执行完后没有再次调用notify方法，则其他线程继续等待。*
若没有在同步代码块或者同步方法中执行wait/notify方法，则会报异常IllegalMonitorStateException,它是RuntimeExceotion的一个子类。
3. 一句话总结wait与notify:
**wait()使线程停止运行，notify()使等待的线程继续运行**。
4. 简单示例:
线程A：
		public class Thread3_02_3A extends Thread{
			
			private Object lock;   //使用Object可以传入任何类型的对象监视器
			public Thread3_02_3A(Object lock) {
				super();
				this.lock=lock;
			}
			@Override
			public void run() {
				super.run();
				try{
					synchronized (lock) {
						lock.wait();             //等待
						System.out.println("wait后的语句");
					}
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}
线程B:
		public class Thread3_02_3B extends Thread{
		
			private Object lock;
			public Thread3_02_3B(Object lock) {
				super();
				this.lock=lock;
			}
			
			@Override
			public void run() {
					synchronized (lock) {
						lock.notify();		//通知
						System.out.println("notify后的语句");
					}
			}
		}
测试类:
		public class Test3_02_3 {
			//notify使等待的线程继续运行wait后的代码
			public static void main(String[] args) {
			   try{
				   Object lock=new Object();    //保证AB线程同步的对象是同一个
				   Thread3_02_3A t1=new Thread3_02_3A(lock);
				   t1.start();
				   Thread.sleep(3000);
				   Thread3_02_3B t2=new Thread3_02_3B(lock);
				   t2.start();
			   }catch (InterruptedException e) {
				   e.printStackTrace();
			   }
			}
		}
测试结果:
		notify后的语句
		wait后的语句

---
## 2.等待/通知机制的实现（用法）
1. 方法wait()释放锁，方法notify()不释放锁
执行wait()方法后锁立即释放，执行notify()方法后锁不会立即释放：
公共资源类:
		public class Object3_03_1 {
			//方法wait()锁释放
			public void methodA(Object lock){
				try{
					synchronized (lock) {
						System.out.println("开始等待");
						lock.wait();		//去掉wait就变成同步了
						Thread.sleep(1000);
						System.out.println("结束等待");
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
两个线程同时访问公共类的methodA()方法，并且传入相同的Object类锁。
若加上wait则两线程异步，若去掉wait则两线程同步。说明**执行wait后立即释放锁**。
公共类:
		public class Object3_03_1 {
			//方法wait()锁释放
			public void methodA(Object lock){
				try{
					synchronized (lock) {
						System.out.println("开始等待");
						lock.wait();		//去掉wait就变成同步了
						Thread.sleep(10000);
						System.out.println("结束等待");
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//方法notify()锁不释放
			public void methodB(Object lock){
				try{
					synchronized (lock) {
						System.out.println("开始唤醒");
						lock.notify;		
						Thread.sleep(10000);
						System.out.println("结束等待");
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
若两线程分别访问methodA与methodB方法,使用同一个Object对象锁，A线程先启动，则产生的结果为:
		开始等待
		开始唤醒
		结束唤醒
		结束等待
说明**执行notify后不立即释放锁**。
2. 在wait时被interrupt中断
当线程呈wait状态时，调用线程对象的**interrupt**方法会出现InterruptedException异常。
3. **notify()只通知并唤醒一个线程**
可以在执行唤醒的线程的run()方法中这样写:
		public void run() {
			super.run();
			synchronized (lock) {
				lock.notify();
				lock.notify();                //连续使用通知多个线程
				lock.notify();
				lock.notify();
			}
		}
实现通知多个线程。
4. **notifyAll()通知并唤醒所有线程**
可以将上述代码修改成这样:
		public void run() {
			super.run();
			synchronized (lock) {
				lock.notifyAll();
			}
		}
实现通知并唤醒所有线程。
5. wait(long)方法的使用:
带一个参数的wait(long)的功能是等待某一时间(long毫秒)内是否有线程对锁进行唤醒，如果*超过这个时间则自动唤醒。*
简单用法：
		synchronized (lock) {
						lock.wait(2000);		//等待2秒
		}
6. 如果通知过早，则wait(）永远不会被唤醒
7. 如果等待wait的条件发生了变化也会造成程序的混乱:
加减方法:
		public class AddCut{
			private String lock;
			public AddCut(String lock){
				super();
				this.lock=lock;
			}
			public void Subtract(){
				try{
					synchronized (lock) {
					if(Object3_08.list.size()==0){
						System.out.println("线程开始等待："+Thread.currentThread().getName());
						lock.wait();
						System.out.println("线程结束等待："+Thread.currentThread().getName());
					}
					Object3_08.list.remove(0);
					System.out.println("list size="+Object3_08.list.size());
					}
				}catch (InterruptedException e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}
			public void add(){
				synchronized (lock) {
				    Object3_08.list.add("6666666");
				    lock.notifyAll();
				}
			}
		}
公共资源类：
		public class Object3_08 {
			public static List list=new ArrayList();
		}
两个线程类:
		public class Thread3_08A extends Thread{
			private AddCut a;
			public Thread3_08A(AddCut a){
				this.a=a;
			}	
			@Override
			public void run() {
				super.run();
				a.add();
			}
		}
		public class Thread3_08B extends Thread{
			private AddCut b;
			
			public Thread3_08B(AddCut b){
				this.b=b;
			}
			@Override
			public void run() {
				super.run();
				b.Subtract();
			}
		}
测试类:
public class Test3_08 {
	public static void main(String[] args) throws InterruptedException {
		String lock=new String("");
		AddCut ac=new AddCut(lock);
		Thread3_08B t1=new Thread3_08B(ac);
		t1.setName("减操作1");
		t1.start();
		Thread3_08B t2=new Thread3_08B(ac);
		t2.setName("减操作2");
		t2.start();
		Thread.sleep(1000);
		Thread3_08A t3=new Thread3_08A(ac);
		t3.setName("加操作1");
		t3.start();
	}
}
输出结果:
		线程开始等待：减操作1
		线程开始等待：减操作2
		线程结束等待：减操作2
		Exception in thread "减操作1" list.size=0
		线程结束等待：减操作1
		java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
			at java.util.ArrayList.rangeCheck(ArrayList.java:653)
			at java.util.ArrayList.remove(ArrayList.java:492)
			at Chapter3.Object.Substract3_08.Subtract(Substract3_08.java:21)
			at Chapter3.Thread.Thread3_08B.run(Thread3_08B.java:16)
原因:
进入等待时list.size==0,但是加后唤醒改变了条件继续执行wait后方法，list长度不足，导致越界异常。
修改AddCut方法:
		public void Subtract(){
			try{
				synchronized (lock) {
				while(Object3_08.list.size()==0){
					System.out.println("线程开始等待："+Thread.currentThread().getName());
					lock.wait();
					System.out.println("线程结束等待："+Thread.currentThread().getName());
				}
				Object3_08.list.remove(0);
				System.out.println("list size="+Object3_08.list.size());
				}
			}catch (InterruptedException e) {
				// TODO: handle exception
				e.printStackTrace();
			}
		}
将if(Object3_08.list.size()==0）{}判断改为while(Object3_08.list.size()==0){}判断，运行结果:
		线程开始等待：减操作1
		线程开始等待：减操作2
		线程结束等待：减操作2
		list size=0
		线程结束等待：减操作1
		线程开始等待：减操作1

---
## 3.生产者消费者模式 
[生产者/消费者模式](https://baike.baidu.com/item/%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E6%A8%A1%E5%BC%8F/15738358?fr=aladdin)是等待/通知模式的经典案例
1. 一生产者与一消费者:操作值
生产者类:
		public class P3_09 {
			//生产者
			private String lock;
			public P3_09(String lock){
				super();
				this.lock=lock;
			}
			public void setValue(){
				try{
					synchronized (lock) {
					    if(!ValueObject3_09.value.equals("")){
					    	lock.wait();
					    }
					    
					    String value=System.currentTimeMillis()+"_"+System.nanoTime();
					    System.out.println("set 的值是"+value);
					    ValueObject3_09.value=value;
					    lock.notify(); 
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
消费者类:
		public class C3_09 {
			//消费者
			private String lock;
			public C3_09(String lock){
				super();
				this.lock=lock;
			}
			public void getValue(){
				try{
					synchronized (lock) {
					    if(ValueObject3_09.value.equals("")){
					    	lock.wait();
					    }
					    System.out.println("get 的值是"+ValueObject3_09.value);
					    ValueObject3_09.value="";
					    lock.notify(); 
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
设置公共资源类（物品类）:
		public class ValueObject3_09 {
			//中间值
			//物品，同一时刻只能有一个生产者或者消费者操作它
			public static String value="";
		}
设置两个线程分别循环执行c.getValue()和p.setValue()方法，测试类只创建两个线程对象（生产者消费者各一个）。
则同一时间只有一个生产者或者消费者对产品操作。value==0时，无产品，消费者等待；value！=0时，产品未被消耗，生产者等待；
2. 多个生产者与多个消费者：操作值，**出现假死**
生产者消费者类如上。测试类中创建多个消费者与多个生产者:
		Thread3_10A[] listp=new Thread3_10A[2];  //生产者
		Thread3_10B[] listc=new Thread3_10B[2];  //消费者
		for(int i=0;i<2;i++){
			listp[i]=new Thread3_10A(p);
			listp[i].setName("生产者"+(i+1));
			listc[i]=new Thread3_10B(c);
			listc[i].setName("消费者"+(i+1));
			listp[i].start();
			listc[i].start();
		}
		//代码不完整，仅供理解
**[假死](http://blog.csdn.net/x_panda/article/details/17200853)**:*所有线程都处在等待状态*。
如这四个线程对象：生产者1，2；消费者1，2；
出现假死的情况：
a.生产者1先执行，仓库无货，添加，发出通知，但是所有线程都不在等待状态，生产者1进入第二次循环，仓库有货，等待；
b.轮到生产者2也因为仓库有货而等待；
c.消费者1进入循环，消费，发出通知，生产者1被唤醒，进入第二次循环，仓库无货，等待；
d.消费者2进入，仓库无货，等待；
e.生产者1被唤醒进入，生产，发出通知，唤醒生产者2，进入第二次循环，仓库有货，等待；
d.生产者2被唤醒进入，仓库有货，等待。出现假死。
出现假死的原因是不能确保唤醒的是"异类"，生产者能唤醒生产者。
解决方法：将notify换成notifyAll;
3. 一生产者与一消费者：操作栈
使生产者向堆栈List对象中放入数据，消费者从List中取出对象，List最大值为1。
生产者：
		public class P3_11 {
			private MyStack3_11 ms;
			
			public P3_11(MyStack3_11 ms) {
				this.ms=ms;
			}
			
			public void pushService(){
				ms.push();
			}
		}
消费者:
		public class C3_11 {
		   private MyStack3_11 ms;
			public C3_11(MyStack3_11 ms) {
				this.ms=ms;
			}
			
			public void popService(){
				ms.pop();
			}
		}
公共堆栈:
		public class MyStack3_11 {
			private List list=new ArrayList<>();
			
			synchronized public void push(){             //增加
				try{
					if(list.size()==1){              //改为while循环可以解决条件改变问题
						this.wait();
					}
					list.add("任何字符串"+Math.random());
					this.notify();     //改为notifyAll解决假死
					System.out.println("push="+list.size());
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			synchronized public String pop(){         //减少
				String returnString="";
				try{
					if(list.size()==0){         //改为while循环可以解决条件改变问题
						System.out.println("pop操作中的："+Thread.currentThread().getName()+"线程呈等待状态");
						this.wait();
					}
					returnString=""+list.get(0);
					list.remove(0);
					this.notify();   // //改为notifyAll解决假死
					System.out.println("pop="+list.size());
					
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
				return returnString;
			}
		}
生产者线程：
		public class Thread3_11A extends Thread{
			private P3_11 p;
			public Thread3_11A( P3_11 p) {
				this.p=p;
			}	
			@Override
			public void run() {
				super.run();
				while(true){
			 		p.pushService();
				}
			}
		}
消费者线程：
		public class Thread3_11B extends Thread{
		    private C3_11 c;
		
		    public Thread3_11B(C3_11 c) {
		    	this.c=c;
			}
		    @Override
		    public void run() {
		    	super.run();
		    	while(true){
		    		c.popService();
		    	}
		    }
		}
测试类:
		public class Test3_11 {
			//一生产与一消费：【操作栈】
			//交替输出
			public static void main(String[] args) {
				MyStack3_11 ms=new MyStack3_11();
				P3_11 p=new P3_11(ms);
				C3_11 c=new C3_11(ms);
				Thread3_11A t1=new Thread3_11A(p);
				Thread3_11B t2=new Thread3_11B(c);
				t1.start();
				t2.start();
			}
		}
4. 一生产与多消费:操作栈
创建一个生产者对象和多个消费者对象,需要解决wait条件发生改变的问题，解决方法仍然是把stack中if()判断改为while判断。把notify方法全都改为notifyAll解决假死。
5. 多生产者与一消费者:操作栈
和一对多差不多，创建多个生产者对象和一个消费者对象。其余相同。
6. 多生产者与多消费者：操作栈
创建多个生产者对象与多个消费者对象，只要注意wait条件的改变（使用while），并且把公共堆栈中的notify方法全都改为notifyAll，就不会产生假死。

---
## 4.使用管道进行线程间通信
### a.管道流简介
Java语言中提供了各种各样的输入/输出流Stream，使我们能够更方便地对数据进行操作，其中[管道流](http://blog.csdn.net/qq_15150353/article/details/52716827)（pipeStream）是一种特殊的流，用于在不同线程中**直接传送数据**。
一个线程发送数据到**输出管道**，另一个线程从**输入管道**读取数据。
JDK提供了四个类：
*PipedTnputStream和PipedOutStream
PipedRead和PipedWriter*
### b.字节流
测试PipedInputStream和PipedOutputStream类：
把值写到输出流:

		public class Write3_15 {
			
			public void writeMethod(PipedOutputStream out){
				try{
				   System.out.println("write :");
				   for(int i=0;i<300;i++){
					   String outString=""+(i+1);
					   out.write(outString.getBytes());   //写值到输出流
					   System.out.print(outString);
				   }
				   System.out.println();
				   out.close();
				}catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
把值从输入流读取（输入）:

		public class Read3_15 {
			
			public void readMethod(PipedInputStream input){
				try{
					System.out.println("read :");
					byte[] bytelist=new byte[20];
					int length=input.read(bytelist);
					while(length!=-1){
						String newString=new String(bytelist, 0, length);
						System.out.print(newString);
						length=input.read(bytelist);  //从输入流读取
					}
					System.out.println();
					input.close();		
				}catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
管道输出流线程（数据写入）:

		public class Thread3_15A extends Thread{
			//管道输出流线程
			private Write3_15 write;
			private PipedOutputStream out;
			
			public Thread3_15A( Write3_15 write ,PipedOutputStream out) {
				this.out=out;
				this.write=write;
			}
			@Override
			public void run() {
				super.run();
				write.writeMethod(out);
			}
		}
管道输入流线程（数据读出）:

		public class Thread3_15B extends Thread{
			//管道输入流线程
			private Read3_15 re;
			private PipedInputStream input;
			
			public Thread3_15B(Read3_15 re,PipedInputStream input) {
				this.re=re;
				this.input=input;
			}
			@Override
			public void run() {
				super.run();
				re.readMethod(input);
			}
		}
测试类:

		public class Test3_15 {
			//结果:输出了两次，一次是input输出的，一次是out输出的
			 public static void main(String[] args) throws InterruptedException {
				try{
					Write3_15 write=new Write3_15();
					Read3_15  red=new Read3_15();
					
					PipedInputStream input=new PipedInputStream();
					PipedOutputStream out=new PipedOutputStream();
					
					//input.connect(out);
					out.connect(input);           //使两个线程产生通信，这样才能将数据进行输入输出
					
					Thread3_15B t1=new Thread3_15B(red, input);
					t1.start();
					Thread.sleep(1000);
					Thread3_15A t2=new Thread3_15A(write, out);
					t2.start();
					
				}catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
测试类中的`input.connect(out);`或`out.connect(input);`作用是使两个线程产生通信，若同时启用这两个语句，会产生:

		java.io.IOException: Already connected
若都关闭，会产生异常:

		Exception in thread "main" java.lang.Error: Unresolved compilation problem: 
			Unreachable catch block for IOException. This exception is never thrown from the try statement body
若先执行读取线程的读取`new Thread3_15B(red, input)`，则线程就会阻塞在`length=input.read(bytelist)`，直到有数据从`out.write(outString.getBytes())`输入，才继续执行。

### c.字符流
PipedRead和PipedWriter的用法与PipedTnputStream和PipedOutStream，下面测试PipedRead和PipedWriter。（传送啥与选择哪对管道流并无关系）
把值写入到输出流:

	public class Write3_16 {
		//输出类
		public void writeMethod(PipedWriter out){
			try{
			   System.out.println("write :");
			   for(int i=0;i<300;i++){
				   String outString=""+(i+1);
				   out.write(outString);
				   System.out.print(outString);
			   }
			   System.out.println();
			   out.close();
			}catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
把值从输入流读出:

	public class Read3_16 {
		//输入类
		
		public void readMethod(PipedReader input){
			try{
				System.out.println("read :");
				char[] bytelist=new char[20];
				int length=input.read(bytelist);
				while(length!=-1){
					String newString=new String(bytelist, 0, length);
					System.out.print(newString);
					length=input.read(bytelist);
				}
				System.out.println();
				input.close();
			}catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
测试类中这样写:

		PipedReader input=new PipedReader();
		PipedWriter out=new PipedWriter();
	  //input.connect(out);
		out.connect(input);

---
## 5.join
1. 使用情况:
主线程创建子线程，若子线程执行大量计算，则主线程会早早结束，如果子线程想取得子线程的值，就要用到join方法，join方法的作用是**等待某对象被销毁**。
如:
		MyThread thread =new MyThread();
		thread.start();
		Thread.sleep(?);
想要让当前线程在thread执行完毕后再执行，不知道sleep中要填多少，这是就能通过join方法来解决：
		MyThread thread =new MyThread();
		thread.start();
		thread.join();
2. join的基本用法:
方法thread.join()的作用是使它所属的线程对象thread正常执行run()方法中的任务，而使当前线程无限阻塞，直到thread对象销毁才继续执行后面的语句。
join具有使线程排队的作用，类似同步，但是join和synchronized的不同是:
**join在内部使用wait()方法进行等待，释放锁；而synchronized关键字使用"对象监视器"原理做为同步**。
具体join的简单用法如上方代码所示。
3. join时碰到interrupt会产生异常:
		java.lang.InterruptedException
			...
但是没有出错的异常会继续执行。
4. join(long)的使用:
内部使用`wait(long)`方法实现，等待线程对象一段时间，若这段时间线程对象未销毁则放弃等待。
join(long)与sleep(long)的区别:
join(long)执行后释放锁，不会立即执行后面的代码，而是等到持有的线程对象执行完销毁或者long时长后再执行;Thread.sleep(long)则不会释放锁，发生阻塞，需要等待long时长后才能执行后面的代码。
5. 使用join时出现的**意外情况**：join后面的代码提前运行。
线程A：
		public class Thread3_23A extends Thread{
			private Thread3_23B tb;
			
			public Thread3_23A(Thread3_23B tb) {
				this.tb=tb;
			}
			
			@Override
			public void run() {
				super.run();
				synchronized (tb) {
					System.out.println("begin A ThreadName="+Thread.currentThread().getName()+" "+System.currentTimeMillis());
					try {
						Thread.sleep(5000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("end A ThreadName="+Thread.currentThread().getName()+" "+System.currentTimeMillis());
				}
			}
		}
线程B:
		public class Thread3_23B extends Thread{
			@Override
			synchronized public void run() {
				super.run();
				try{
					System.out.println("begin b ThreadName="+Thread.currentThread().getName()+" "+System.currentTimeMillis());
					Thread.sleep(5000);
					System.out.println("end b ThreadName="+Thread.currentThread().getName()+" "+System.currentTimeMillis());
				}catch (InterruptedException e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}
		}
测试类:
		public class Test3_23 {	
			/**
			 * join再次得到锁时已经过了两秒，自动释放导致失效
			 * join总是最先抢到锁
			 */
			public static void main(String[] args) {
				try{
					Thread3_23B tb=new Thread3_23B();
					Thread3_23A ta=new Thread3_23A(tb);
					ta.start();
					tb.start();
					tb.join(2000);               //将时间变长或直接用join()方法而不用join(long)可解决
					System.out.println("main 方法结束了");
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		
		}
join总是最先抢到锁。join再次得到锁时已经过了两秒，自动释放线程异步执行导致后面的代码先执行。

---
## 6.ThreadLocal类
1. ThreadLocal简介及简单用法
变量值可以使用public static变量的形式，所有的线程都使用**同一个public static变量**。
类ThreadLocal主要解决的是每个线程绑定自己的值，**存储每个线程的私有变量**。简单用法:
		public class Test{
			public static ThreadLocal t1=new  ThreadLocal();
			
			public static void main(String[] args) {
				if(t1.get()==null){
					System.out.println(t1.get());
					System.out.println("此时ThreadLocal中没有值");
					t1.set("有值了");
				}
				System.out.println(t1.get());
				System.out.println(t1.get());
			}
		}
默认初始值为null，通过set()赋值与get()取值，取值后值不消失。
2. 变量的隔离性：
公共资源类:
		public class Object3_25_1{
			//验证线程变量的隔离性
			public static ThreadLocal t1=new ThreadLocal();
		}
线程A:
		public class Thread3_25_1A extends Thread{
			//验证线程变量的隔离性
			@Override
			public void run() {
				super.run();
				try{
					Object3_25_1.t1.set("ThreadA的值");
					System.out.println("ThreadA取出的值为:"+Object3_25_1.t1.get());
					Thread.sleep(200);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
线程B：
		public class Thread3_25_1B extends Thread{
			//验证线程变量的隔离性
			@Override
			public void run() {
				super.run();
				try{
					for(int i=0;i<100;i++){
						Object3_25_1.t1.set("ThreadB的值");
						System.out.println("ThreadB取出的值:"+Object3_25_1.t1.get());
						Thread.sleep(200);
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
测试类:
		public class Test3_25_1 {
			//每个线程都能取出自己的值
			public static void main(String[] args) {
				try{
					Thread3_25_1A ta=new Thread3_25_1A();
					Thread3_25_1B tb=new Thread3_25_1B();
					ta.start();
					tb.start();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
运行结果:
		ThreadA的值
		ThreadA取出的值为：ThreadA的值
		ThreadB的值
		ThreadB取出的值为：ThreadB的值
3. 设置初始值，使它不为null
ThreadLocal继承类:
		public class ThreadLocalExt extends ThreadLocal{
			//解决get返回初始值为null的问题
			//继承至ThreadLocal的initialValue()方法
			protected Object initialValue(){
				return "这是默认值，不再是null了";
			}
		}
修改公共资源类:
		public class Object3_25_1{
			public static ThreadLocalExt t1=new ThreadLocalExt();
		}
就可以设置并使用默认初始值了。

---
## 7.InheritableThreadLocal类
1. 使用InheritableThreadLocal可以在子线程中取得父线程中设置的值(父线程是指启动子线程的线程)
设置初始值类:
		public class InheritableThreadLocalExt extends InheritableThreadLocal<Object>{
			//InheritableThreadLocal值继承
			protected Object initialValue(){
				return new Date().getTime();
			}
		}
公共资源类:
		public class Object3_28_2 {
			public static InheritableThreadLocalExt t=new InheritableThreadLocalExt();
		}
线程类:
		public class Thread3_28 extends Thread{
			//InheritableThreadLocal值继承
			@Override
			public void run() {
				super.run();
				try{
					System.out.println("子线程得到的值："+Object3_28_2.t.get());
					Thread.sleep(100);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
测试类(主线程):
		public class Test3_28 {
			//InheritableThreadLocal值继承
			public static void main(String[] args) {
				try{
					System.out.println(" 在父线程中得到的值="+Object3_28_2.t.get());
					Thread.sleep(100);
					Thread.sleep(5000);
					Thread3_28 tb=new Thread3_28();
					tb.start();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
得到的两个输出值相同，说明子线程是从父线程得到的值，因为不是的话，因为Date的原因，不能输出相同的值：
		在父线程中得到的值=1520064921970
		子线程得到的值：1520064921970
2. 修改继承下来的值
修改设置初始值类:
		public class InheritableThreadLocalExt extends InheritableThreadLocal{
			protected Object initialValue(){
				return new Date().getTime();
			}
			
			protected Object childValue(Object parentValue){
				return parentValue+" 在子线程添加的";
			}
		}
则子类会在取得父类值的基础上加上"在子线程添加的":
		在父线程中得到的值=1520064869836
		子线程得到的值：1520064869836 在子线程添加的
3. 如果在子线程得到值的同时,主线程对InheritableThreadLocal中的值进行更改那么子线程得到的值还是旧值

---