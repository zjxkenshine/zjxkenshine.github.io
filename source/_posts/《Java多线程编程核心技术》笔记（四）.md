---
title: 《Java多线程编程核心技术》笔记（四）:Lock的使用
date: 2018-03-03 16:17:16
tags:
- Java
- 多线程
categories: 学习笔记

---
## 0.学习要点
- ReentrantLock类的使用。
- ReentrantLock常用方法
- ReentranReadWritetLock类的使用。

---
## 1.ReentratLock类
1. 简介：
JDK1.5中新增了ReentrantLock类可以和synchronized类一样达到同步效果，并且在扩展功能上**更加强大**，如有嗅探锁定、多路分支通知等功能，而且在使用上比synchronized更加灵活。
调用ReentrantLock的lock()方法获取锁，调用unlock()方法释放锁。
2. 使用
简单语法:
		private Lock lock=new ReentrantLock();
在公共资源类中这样使用：
		public class Service{
			//使用ReentratLock实现同步：测试1
			private Lock lock=new ReentrantLock();
			
			public void testMethod(){
				lock.lock();             //锁开始
				//需要同步的代码块
				lock.unlock();           //释放锁
			}
		}
线程类：
		public class Thread4_01 extends Thread{
			//使用ReentratLock实现同步：测试1
			private Service ser=new Service();
			
			public Thread4_01(Service ser) {
				this.ser=ser;
			}
			
			@Override
			public void run() {
				super.run();
				ser.testMethod();
			}
		}
若有Thread.sleep()方法，公共资源类可以这样写:
		public class Service {
			private Lock lock=new ReentrantLock();
			
			public void MethodA(){
				try{
					lock.lock();
					//公共代码块
					Thread.sleep(2000);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
		}
若公共资源类这样写:
		public class Service {
			private Lock lock=new ReentrantLock();
			
			public void MethodA(){
				lock.lock();
				//公共代码块
				lock.unlock();
			}
			
			public void MethodB(){
				lock.lock();
				//公共代码块
				lock.unlock();
			}
		}
访问方法A与方法B是同步的。

---
## 2.Lock结合Condition实现等待/通知
1. 简介:
synchronized与wait()、notify()/notifyAll()方法结合使用可以实现等待/通知模式，ReentrantLock也可以实现同样的功能，但要借助**Condition对象**。
Condition是JDK5中出现的，使用它有更好的灵活性，如可以实现多路通知功能，也就是**在一个Lock对象里可以创建多个Conditon(即对象监视器)实例**，线程对象可以注册在指定的Condition中，从而可以有选择性地进行通知，调度上更加灵活。
**synchronized相当与一个Lock对象中只有一个Condition对象**，所有线程都注册在一个对象身上，使用notifyAll()方法时会通有等待线程，没有选择权，会出现效率问题。
2. 使用:
简单语法:
		private Lock lock=new ReentrantLock();
		public Condition con=lock.newCondition();
		try{
			con.await();
			con.signal();
		}catch(InterruptedException e){
			e.printStackTrace();
		}
错误用法:
		public class Service4_03_1 {
			//使用Condition实现等待/通知：错误的用法及解决
			private Lock lock=new ReentrantLock();
			private Condition con=lock.newCondition();
			
			public void await(){
				try{
					con.await();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
会出错：
		Exception in thread "Thread-0" Exception in thread "Thread-0" java.lang.IllegalMonitorStateExceptionjava.lang.IllegalMonitorStateException
正确用法：
在`con.await()`前调用`lock.lock();`获得同步监视器:
		public class Service4_03_2 {
			private Lock lock=new ReentrantLock();
			private Condition con=lock.newCondition();
			//结果只输出了"11111111111111111"
			public void await(){
				try{
					lock.lock();
					System.out.println("11111111111111111");
					con.await();
					System.out.println("22222222222222222");
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally{
					lock.unlock();
					System.out.println("锁释放了");
				}
			}
		}
没有线程使用signal()或者signalAll()方法唤醒await()的condition对象，则结果只会输出"11111111111111111"。
3. Condition实现等待通知：
公共资源类:
		public class Service4_04 {
			private Lock lock=new ReentrantLock();
			public Condition con=lock.newCondition();
			
			//等待方法
			public void await(){
				try{
					lock.lock();
					System.out.println("等待时间为"+System.currentTimeMillis());
					con.await();
					
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally{
					lock.unlock();
				}
			}	
			//通知方法
			public void signal(){
				try{
					lock.lock();
					System.out.println("通知时间为"+System.currentTimeMillis());
					con.signal();
				}finally {
					lock.unlock();
				}
			}
		}
线程类：
		public class Thread4_04 extends Thread{
			private Service4_04 ser=new Service4_04();
			
			public Thread4_04(Service4_04 ser) {
				this.ser=ser;
			}
			
			@Override
			public void run() {
				super.run();
				ser.await();
			}
		}
测试类：
		public class Test4_04 {
			public static void main(String[] args) throws InterruptedException {
				Service4_04 ser=new Service4_04();
				Thread4_04 th=new Thread4_04(ser);
				th.start();
				Thread.sleep(3000);
				ser.signal();
			}
		}
测试结果:
		等待时间为1520086544293
		通知时间为1520086547297
Object类中的wait()方法相当于Condition类中的await()方法
Object类中的wait(long timeout)方法相当于Condition类中的await(long time,TimeUnit unit)方法
Object类中的notify()方法相当于Condition类中的signal()方法
Object类中的notifyAll()方法相当于Condition类中的signalAll()方法
4. 使用**多个condition实现通知多个线程**
单独唤醒部分线程:
使用多个Condition对象，使用condition对象可以唤醒部分指定线程。可以先对线程进行分组，然后唤醒指定组中的线程。
公共资源类:
		public class Service4_06 {
			private Lock lock=new ReentrantLock();
			public Condition conditionA=lock.newCondition();
			public Condition conditionB=lock.newCondition();
			//ConditionA等待
			public void awaitA(){
				try{
					lock.lock();
					System.out.println("begin awaitA的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
					conditionA.await();
					System.out.println("end awaitA的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally{
					lock.unlock();
				}
			}
			//ConditionB等待
			public void awaitB(){
				try{
					lock.lock();
					System.out.println("begin awaitB的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
					conditionA.await();
					System.out.println("end awaitB的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally{
					lock.unlock();
				}
			}
			//ConditionA通知
			public void signalA(){
				try{
					lock.lock();
					System.out.println("signalA begin的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
					conditionA.signal();
				}finally {
					lock.unlock();
				}
			}
			//ConditionB通知
			public void signalB(){
				try{
					lock.lock();
					System.out.println("signalBbegin的时间为"+System.currentTimeMillis()+" ThreadName="+Thread.currentThread().getName());
					conditionA.signal();
				}finally {
					lock.unlock();
				}
			}
		}
线程A：
		public class Thread4_06A extends Thread{
			private Service4_06 ser=new Service4_06();
			
			public Thread4_06A(Service4_06 ser){
				super();
				this.ser=ser;
			}
			
			@Override
			public void run() {
				super.run();
				ser.awaitA();
			}
		}
线程B:
		public class Thread4_06B extends Thread{
			private Service4_06 ser=new Service4_06();
			
			public Thread4_06B(Service4_06 ser){
				super();
				this.ser=ser;
			}
			
			public void run() {
				super.run();
				ser.awaitB();
			}
		}
测试类:
		public class Test4_06 {
			public static void main(String[] args) throws InterruptedException {
				Service4_06 ser=new Service4_06();
				Thread4_06A a=new Thread4_06A(ser);
				a.setName("A");
				a.start();
				Thread4_06B b=new Thread4_06B(ser);
				b.setName("B");
				b.start();
				Thread.sleep(3000);
		
				ser.signalA();           //仅唤醒A
			//	ser.signalB();            //仅唤醒B
			}
		}
测试结果:
		begin awaitA的时间为1520087906578 ThreadName=A
		begin awaitB的时间为1520087906578 ThreadName=B
		signalA begin的时间为1520087909582 ThreadName=main
A线程与B线程是同步访问lock.lock()和lock.unlock()包裹中的代码的。

---
## 3.Lock结合Condition实现生产者/消费者模式
1. 一对一交替打印:
公共资源类:
		public class Service4_07 {
		  //实现生产者/消费者模式：一对一打印
			private ReentrantLock lock=new ReentrantLock();
			private Condition condition=lock.newCondition();
			private boolean value=false;    //仓库，中间值
			public void set(){
				try{
					lock.lock();
					while(value==true){
						condition.await();
					}
					System.out.println("打印11111111111");
					value=true;
					condition.signal();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
			public void get(){
				try{
					lock.lock();
					while(value==false){
						condition.await();
					}
					System.out.println("打印22222222222");
					value=false;
					condition.signal();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
		}
测试类:
		public class Test4_07 {
			//实现生产者/消费者模式：一对一打印
			public static void main(String[] args) {
				Service4_07 ser=new Service4_07();
				Thread4_07A t1=new Thread4_07A(ser);  //循环执行set()方法的线程
				t1.start();
				Thread4_07B t2=new Thread4_07B(ser);  //循环执行get()方法的线程
				t2.start();
			}
		}
测试结果:
		打印11111111111
		打印22222222222
		打印11111111111
		打印22222222222
		打印11111111111
		打印22222222222
		打印11111111111
		.....
2. 多生产者多消费者：多对多打印
将公共资源类修改如下:
		public class Service4_08 {	
			private ReentrantLock lock=new ReentrantLock();
			private Condition condition=lock.newCondition();
			private boolean value=false;
			
			public void set(){
				try{
					lock.lock();
					while(value==true){
						condition.await();
					}
					System.out.println("打印11111111111");
					value=true;
		//			condition.signal();            //会假死
					condition.signalAll();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
			
			public void get(){
				try{
					lock.lock();
					while(value==false){
						condition.await();
					}
					System.out.println("打印22222222222");
					value=false;
				//	condition.signal();            //会假死
					condition.signalAll();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
		}
测试类修改:
		public class Test4_08 {
			public static void main(String[] args) {
				Service4_08 ser=new Service4_08();
				Thread4_08A[] t1=new Thread4_08A[10];
				Thread4_08B[] t2=new Thread4_08B[10];
				for(int i=0;i<10;i++){
					t1[i]=new Thread4_08A(ser);  //循环执行set()方法的线程
					t2[i]=new Thread4_08B(ser);  //循环执行get()方法的线程
					t1[i].start();
					t2[i].start();
				}
			}
		}
运行结果可能会出现连续打印1111111或者连续打印222222的情况，原因是只使用了一个Condition对象并使用signalAll()唤醒。使用两个condition对象并分组唤醒就可以实现交替打印了，也不会产生死锁。

---
## 4.公平锁与非公平锁
1. 简介
公平锁:
*表示线程获取锁的顺序是按线程加锁(start)的顺序来分配的，即先来先得FIFO顺序。*
非公平锁:
*一种获取锁的抢占机制，是随机获得锁，先来的不一定拿到锁，可能造成某些线程一直得不到锁。*
简单用法:
		ReentrantLock lock=new ReentrantLock(true/false);
true是公平锁，false是非公平锁。默认值是false。
2.使用:
在公共资源中这样写:
		public class Service4_09 {
			//公平锁与非公平锁
			private ReentrantLock lock;
			
			public Service4_09(boolean isfair) {
				super();
				lock=new ReentrantLock(isfair);
			}
			
			public void Method(){
				try{
					lock.lock();
					System.out.println("ThreadName="+Thread.currentThread().getName()+" 得到锁");
				}finally {
					lock.unlock();
				}
			}
		}

---
## 5.ReentrantLock类的一些基本方法
### 5.1 getHoldCount(),getQueueLength()和getWaitQueueLenth(Condition)方法
1. int getHoldCount()方法:
*查询当前线程保持此锁的个数，也就是调用lock()方法的次数(重入锁的次数+1)。*
		lock.getHoldCount();
公共资源类:
		public class Service4_10_1 {
			//getHoldCount()方法
			private ReentrantLock lock=new ReentrantLock();
			public void Method1(){
				try{
					lock.lock();
					System.out.println("Method1 getHoldCount="+lock.getHoldCount());  //getHoldCount()方法
					Method2();
				}finally {
					lock.unlock();
				}
			}
		
			private void Method2() {
				try{
					lock.lock();
					System.out.println("Method2 getHoldCount="+lock.getHoldCount());  //getHoldCount()方法
					Method3();
				}finally {
					lock.unlock();
				}
				
			}
		
			private void Method3() {
				try{
					lock.lock();
					System.out.println("Method3 getHoldCount="+lock.getHoldCount());
				}finally {
					lock.unlock();
				}
			}
		}
在测试类中调用Method1方法，运行结果为:
		Method1 getHoldCount=1
		Method2 getHoldCount=2
		Method3 getHoldCount=3
2. int getQueueLength()方法：
*返回正等待获取此锁定的线程估计数。*
如有5个线程等待lock锁，1个线程首先执行await()方法，在其余4个线程未执行lock()前调用lock.getQueueLength()返回值为4。
		lock.getQueueLength();
公共资源类:
		public class Service4_10_2 {
			public ReentrantLock lock=new ReentrantLock();
			public void TestMethod(){
				try{
				    lock.lock();
				    System.out.println("ThreadName="+Thread.currentThread().getName()+" 进入方法");
				    Thread.sleep(Integer.MAX_VALUE);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
		}
测试类:
		public class Test4_10_2 {
			//getQueueLength()方法	
			public static void main(String[] args) throws InterruptedException {
				final Service4_10_2 ser=new Service4_10_2();
				Runnable runn=new Runnable() {	
					@Override
					public void run() {
						ser.TestMethod();
						
					}
				};
				Thread[] tl=new Thread[10];
				for(int i=0;i<10;i++){
					tl[i]=new Thread(runn);
				}
				for(int i=0;i<10;i++){
					tl[i].start();
				}
				Thread.sleep(2000);
				System.out.println("有"+ser.lock.getQueueLength()+"个线程在等待锁");
			}
		}
测试结果:
		ThreadName=Thread-1 进入方法
		有9个线程在等待锁
3. int getWaitQueueLength(Condition condition)方法
*返回等待与此锁定相关的给定条件Condition的线程估计数。*
如有5个线程，每一个线程都执行了同一个Condtion对象的await()方法，则调用该方法返回值为5。
		lock.getWaitQueueLength(condition)；
公共资源类:
		public class Service4_10_3 {
			//getWaitQueueLength()方法
			private ReentrantLock lock=new ReentrantLock();
			private Condition con=lock.newCondition();
			
			public void WaitMethod(){
				try{
					lock.lock();
					con.await();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
			
			public void SingalMethod(){
				try{
					lock.lock();
					System.out.println("有"+lock.getWaitQueueLength(con)+"个线程正在等待con");
					con.signal();
				}finally {
					lock.unlock();
				}	
			}
		}
测试类调用SingalMethod（）方法时可以打印出共有多少个线程在等待。

### 5.2 hasQueueThread(Thread),hasueueThreads()和hasWaiters(Condition)方法
1. boolean hasQueueThread(Thread thread)方法:
*查询指定线程是否正在等待此锁定释放。*
		lock.hasQueueThread(thread);
2. boolean hasQueueThreads()方法:
*查询有线程是否正在等待获取此锁定。*
		lock.hasQueueThreads();
3. boolean hasWaiters(Condition condition)方法:
*查询是否有线程正在等待（wait）与此锁定condition有关的通知（notify）。*
		lock.hasWaiters(condition);
而lock.getWaitQueueLength(condition)是查询有多少个正在等待:
公共资源类:
		public class Service4_11_2 {
			public ReentrantLock lock=new ReentrantLock();
			public Condition con=lock.newCondition();
			
			public void waitMethod(){
				try{
					lock.lock();
					con.await();
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally{
					lock.unlock();
				}
			}
			
			public void notifyMethod(){
				try{
					lock.lock();
					System.out.println("有么有正在等待该condition对象的线程："+lock.hasWaiters(con));
					System.out.println("有多少个线程在等待："+lock.getWaitQueueLength(con));
					con.signal();
				}finally {
					lock.unlock();
				}
			}
		}

### 5.3 isFair(),isHeldByCurrentThread()和isLocked()方法
1. boolean isFair()方法:
*判断该锁是不是公平锁。*
		lock.isFair();
2. boolean isHeldByCurrentThread()方法:
*查询当前线程是否保持此锁定。*
		lock.isHeldByCurrentThread();
3. boolean isLocked()方法:
*查询此锁定是否由任意线程保持。*
		lock.isLocked();
公共资源类:
		public class Service4_12_3 {
			private ReentrantLock lock=new ReentrantLock();
			
			public Service4_12_3(boolean isFair) {
				lock=new ReentrantLock(isFair);
			}
			
			public void TestMethod(){
				try{
					System.out.println(lock.isLocked());
					System.out.println(lock.isHeldByCurrentThread());
					lock.lock();
					System.out.println(lock.isLocked());
					System.out.println(lock.isHeldByCurrentThread());
				}finally {
					lock.unlock();
					System.out.println(lock.isLocked());
					System.out.println(lock.isHeldByCurrentThread());
				}
			}
		}
在测试类中使用线程对象执行TestMethod()方法,结果为:
		false
		false
		true
		true
		false
		false

### 5.4 lockInterruptibly(),tryLock()和tryLock(long timeout,TimeUnit unit)方法
1. void lockInterruptibly()方法:
*如果当前线程未被中断，则获取锁定，如果已经被中断，则出现异常。*
lock遇到interrupt不会被中断。lockInterruptibly遇到interrupt会被异常。
		lock.lockInterruptibly();
公共资源类:
		public class Service{
			public ReentrantLock lock=new ReentrantLock();
			private Condition con=lock.newCondition();
			
			public void TestMethod() throws InterruptedException{
				try{
					//lock.lock();
					lock.lockInterruptibly();
					for(int i=0;i<Integer.MAX_VALUE/10;i++){
						String newString=new String("");
						Math.random();
					}
				}finally {
					if(lock.isHeldByCurrentThread()){
						lock.unlock();
					}
				}
			}
		}
测试类:
		public static void main(String[] args) throws InterruptedException {
			final Service ser=new Service();
			Runnable run=new Runnable() {
				
				@Override
				public void run() {
					try {
						ser.TestMethod();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			};
			Thread t2=new Thread(run);
			t2.setName("BBB");
			t2.start();
			t2.interrupt();   //b线程中断时会报错
			System.out.println("main end");
		}
2. boolean tryLock()方法:
*仅在调用时锁定未被另一个线程保持的情况下，才获取该锁定。并返回一个布尔值。*
		lock.tryLock();
公共资源类:
		public class Service4_13_2 {
			//tryLock()方法
			public ReentrantLock lock=new ReentrantLock();
			public void Method(){
				if(lock.tryLock()){
					System.out.println(Thread.currentThread().getName()+" 获得锁");
				}else{
					System.out.println(Thread.currentThread().getName()+" 没有获得锁");
				}
			}
		}
3. boolean tryLock(long timeout,TimeUnit unit)方法:
*如果锁定在给定时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁定;若当前线程被中断，则报`java.lang.InterruptedException`异常。*
timeout：数字，多长时间
TimeUnit:时间单位
		lock.tryLock(3,TimeUnit.SECONDS);
公共资源类:
		public class Service{
			//tryLock(long timeout,TimeUnit unit)
			public ReentrantLock lock=new ReentrantLock();
			public void testMethod(){
				try{
					if(lock.tryLock(3, TimeUnit.SECONDS)){
						System.out.println("     "+Thread.currentThread().getName()+" 获得锁的时间"+System.currentTimeMillis());
			         	Thread.sleep(10000);//改为3秒以内且线程未中断可以获得锁
					}else{
						System.out.println("     "+Thread.currentThread().getName()+"没有获得锁");
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					if(lock.isHeldByCurrentThread()){
						lock.unlock();
					}
				}
			}
		}
### 5.4 awaitUninterruptibly()和awaitUntil()方法
1. void awaitUninterruptibly()方法:
*作用与await相同，但是遇到interrupt时不会报错。*
		condition.awaitUninterruptibly();
而且不需要try/catch包裹:
		public void testMethod(){
			try{
				lock.lock();
				System.out.println("wait begin");
				con.awaitUninterruptibly();
				System.out.println("wait end");
			}finally {
				lock.unlock();
			}
		}
2. void awaitUntil(Date deadline)方法:
*使线程等待到某一时刻，但是提前可以被提前强制唤醒。*
		lock.awaitUntil(deadline);
公共资源类:
		public class Service4_15 {
			//方法awaitUntil()的使用
			private ReentrantLock lock=new ReentrantLock();
			private Condition con=lock.newCondition();
			
			public void waitMethod(){
				try{
					Calendar cal=Calendar.getInstance();
					cal.add(Calendar.SECOND, 10);
					
					lock.lock();
					System.out.println("wait begin time="+System.currentTimeMillis());
					con.awaitUntil(cal.getTime());
					
					System.out.println("wait end time="+System.currentTimeMillis());
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.unlock();
				}
			}
			
			public void notifyMethod(){
				try{
					Calendar cal=Calendar.getInstance();
					cal.add(Calendar.SECOND, 10);
					
					lock.lock();
					System.out.println("notify begin time="+System.currentTimeMillis());
					con.signalAll();
					System.out.println("notify end time="+System.currentTimeMillis());
				}finally {
					lock.unlock();
				}
			}
		}

---
## 6.ReentrantReadWriteLock类
1. 读写锁简介:
类ReentrantLock具有完全互斥排他的效果，即同一时间只有一个线程在执行ReentrantLock.lock()方法后面的任务。这样做虽然保证了实例变量的安全性，效率却非常低下。JDK中提供了一种读写锁ReentrantReadWriteLock()来提升代码的运行效率。
**[读写锁](https://baike.baidu.com/item/%E8%AF%BB%E5%86%99%E9%94%81/1756708?fr=aladdin)**表示有两个锁，一个与读相关的锁，称为共享锁，另一个是和写相关的锁，称为排他锁。**多个读锁不互斥，读锁与写锁互斥，写锁与写锁互斥**。没有线程进行写操作时，进行读取的多个线程都可以获得读锁，一旦有线程进行写入，读写，写读，写写就会产生互斥。多个线程可以同时进行读操作，但是同一时间只允许一个线程进行写操作。
2. 读锁:
		private ReentrantReadWriteLock lock=new ReentrantReadWriteLock();
		lock.readLock().lock();
		lock.readLock().unlock();
写锁:
		private ReentrantReadWriteLock lock=new ReentrantReadWriteLock();
		lock.writeLock().lock();
		lock.writeLock().unlock();
3. 代码示例:
公共资源类:
		public class Service4_19 {
			private ReentrantReadWriteLock lock=new ReentrantReadWriteLock();
			
			public void write(){
				try{
					lock.writeLock().lock();
					System.out.println(Thread.currentThread().getName()+"获得写锁，时间"+System.currentTimeMillis());
					Thread.sleep(2000);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.writeLock().unlock();
				}
			}
			
			public void Read(){
				try{
					lock.readLock().lock();
					System.out.println(Thread.currentThread().getName()+"获得读锁，时间"+System.currentTimeMillis());
					Thread.sleep(2000);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					lock.readLock().unlock();
				}
			}
		}
当有不同线程同时访问同一个lock的读写方法时，会出现互斥，当写方法执行完毕后，读方法是异步执行的。

---