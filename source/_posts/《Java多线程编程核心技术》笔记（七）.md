---
title: 《Java多线程编程核心技术》笔记（七）：其他剩余知识点
date: 2018-03-06 13:41:57
tags:
- Java
- 多线程
categories: 学习笔记

---
## 0.学习要点
- 线程的状态与切换
- 线程组的使用
- SimpleDataFormat类与多线程的意外及解决办法
- 如何处理线程的异常

---
## 1.线程的状态
1. 简介:
在java.lang.Thread中线程状态的定义(注释已删除)：
	   public enum State {
	        NEW,  //至今尚未启动的线程的状态
	        RUNNABLE,  //正在虚拟机中执行的线程的状态
	        BLOCKED,  //受阻塞并等待某个监视器锁的状态
	        WAITING,  //无限期地等待另一个线程来执行某一特定操作的线程的状态
	        TIMED_WAITING,  //等待另一个线程来执行操作取决于指定等待时间的线程处于这种状态
	        TERMINATED;  //已退出的线程的状态
	    }
产生各种状态的代码:
**NEW到RUNNABLE**:start()
**RUNNABLE到WATING**:wait()/join()/LockSupport.park()
**RUNNABLE到TIMED_WAITING**:wait(timeout)/sleep(sleeptime)/join(timeout)/LockSupport.parkNanos()/LockSupport.parkUntil()
**RUNNABLE到BLOCK**:线程阻塞
**WATING到RUNNABLE**:notify()/notifyAll()
**除NEW,BLOCK到TERMINATED**:线程停止或退出。
有些状态之间是不能转换的:
NEW只能到RUNNABLE,不能转换为其他状态；
BLOCKED不能转换为TERMINATED；
TERMINATED不能转换为其他状态。
2. 查看线程状态：
		thread.getState();
3. NEW,RUNNABLE和TERMINATED
NEW状态是实例化后未执行run的状态，RUNNABLE是线程进入运行时的状态，TERMINATED状态是线程销毁时的状态
线程类:
		public class Thread7_01 extends Thread{
			//验证NEW,RUNNABLE和TERMINATED状态
			public Thread7_01() {
				System.out.println("构造方法中的状态："+Thread.currentThread().getState());          //这里显示的是main线程的状态
			}
			
			@Override
			public void run() {
				super.run();
				System.out.println("run方法中的状态："+Thread.currentThread().getState());
			}
		}
测试类:
		public static void main(String[] args) {
			try{
				Thread7_01 t1=new Thread7_01();
				System.out.println("main方法中的状态1："+t1.getState());
				Thread.sleep(1000);
				t1.start();
				Thread.sleep(1000);
				System.out.println("main方法中的状态2："+t1.getState());
			}catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
运行结果:
		构造方法中的状态：RUNNABLE
		main方法中的状态1：NEW
		run方法中的状态：RUNNABLE
		main方法中的状态2：TERMINATED
4. TIMED_WAITING状态
执行sleep方法后状态的枚举值就变为了TIMED_WAITING
线程类:
		public class Thread7_02 extends Thread{
			//验证TIMED_WAITING状态
			@Override
			public void run() {
				super.run();
				try{
					System.out.println("begin sleep");
					Thread.sleep(10000);
					System.out.println("end sleep");
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
测试类:
		public static void main(String[] args) {
			try{
				Thread7_02 t1=new Thread7_02();
				t1.start();
				Thread.sleep(1000);
				System.out.println("main方法中的状态"+t1.getState());
			}catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
运行结果:
		begin sleep
		main方法中的状态TIMED_WAITING
		end sleep
5. BLOCKED与WATING:
公共资源类:
		public class Service7_03 {
			//验证BLOCKED状态
			synchronized static public void serviceMethod(){
				try{
					System.out.println(Thread.currentThread().getName()+"进入了业务方法");
					Thread.sleep(5000);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
测试类:
		public static void main(String[] args) throws InterruptedException {
			Thread7_03A t1=new Thread7_03A();  //线程A执行serviceMethod方法
			t1.setName("AAA");
			t1.start();
			Thread.sleep(100);
			Thread7_03B t2=new Thread7_03B();  //线程B执行serviceMethod方法
			t2.setName("BBB");
			t2.start();
			Thread.sleep(100);        //去掉这一行状态变为Runnable
			System.out.println("main方法中的t2的状态："+t2.getState());
			Thread.sleep(5000); 
			System.out.println("main方法中的t1的状态："+t1.getState());
		}
测试结果:
		AAA进入了业务方法
		main方法中的t2的状态：BLOCKED
		BBB进入了业务方法
		main方法中的t1的状态：WAITING

---
## 2.线程组
### 2.1 对象关联线程组
1. 简介：
a.可以把线程归属到某一个线程组中，线程组中可以有线程对象也可以有线程组,组中还可以有线程
b.线程组的作用是，**可以批量管理线程或线程组,有效地对线程或线程组对象进行组织**
Thread的四个与线程组有关的构造方法:
		public Thread(ThreadGroup group, Runnable target);
		public Thread(ThreadGroup group, String name);
		public Thread(ThreadGroup group, Runnable target, String name);
		public Thread(ThreadGroup group, Runnable target, String name, long stackSize);
2. 线程对象关联线程组:**1级关联**
所谓的1级关联就是父对象中有子对象，但是并不创建子孙对象。
这种情况常出现在开发中。通常情况下是创建一个线程组，然后再将部分线程归属到该组中。这样处理可以对零散的线程对象进行有效的组织与规划。
		public static void main(String[] args) {
			Thread7_05A t1=new Thread7_05A();
			Thread7_05B t2=new Thread7_05B();
			ThreadGroup tg=new ThreadGroup("666组");
			
			Thread t11=new Thread(tg,t1);  //将线程对象归并到tg组
			Thread t22=new Thread(tg,t2);
			
			t11.start();
			t22.start();
			
			System.out.println("活动线程数为："+tg.activeCount());
			System.out.println("线程组的名称为："+tg.getName());
		}
tg.activeCount()：得到线程组对象中活动的线程数量。
tg.getName():得到线程组的名称。
3. 线程对象关联线程组:**多级关联**
多级关联就是父对象中有子对象，子对象中再创建子对象，也就是出现子孙对象。
**此种方法在开发中不常见**，线程树过于复杂反而不利于线程对象的管理，但是JDK提供了支持多级关联的线程树结构:
		public static void main(String[] args) {
		    //在main线程组中添加一个线程组tg1,在tg1中添加线程对象Z
			//方法activeGroupCount()和activeCount()的值不是固定的
			//是环境中的一个快照
			ThreadGroup mainGrop=Thread.currentThread().getThreadGroup();  //main线程的线程组
			ThreadGroup tg=new ThreadGroup(mainGrop,"666");  //main组中加666组
			Runnable runn=new Runnable() {
				@Override
				public void run() {
					try {
						System.out.println("runMethod");
						Thread.sleep(10000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			};
			Thread newtd=new Thread(tg,runn);  //z线程归到tg组
			newtd.setName("Z");
			newtd.start();            //必须启动才会归到tg线程组中
			
			ThreadGroup[] listgroup=new ThreadGroup[Thread.currentThread().getThreadGroup().activeCount()];
			Thread.currentThread().getThreadGroup().enumerate(listgroup);
			System.out.println("main线程组有多少个子线程组"+listgroup.length+"   名字为："+listgroup[0].getName());
			
			Thread[] listTd=new Thread[listgroup[0].activeCount()];
			listgroup[0].enumerate(listTd);
			System.out.println(listTd[0].getName());
		}
java.lang.ThreadGroup.enumerate(Thread[] list)方法：*复制该线程组及其子组中的所有活动线程到指定的线程数组。同样适用于线程组数组。*
4. 线程组自动归属特性:
		public static void main(String[] args) {	
			System.out.println("A处线程："+Thread.currentThread().getName()+" 所属线程组名称："+Thread.currentThread().getThreadGroup()+" 线程组内有线程组的数量："+Thread.currentThread().getThreadGroup().activeGroupCount());
			ThreadGroup group=new ThreadGroup("新的组");          //自动加入到main线程组中
			System.out.println("B处线程："+Thread.currentThread().getName()+" 所属线程组名称："+Thread.currentThread().getThreadGroup()+" 线程组内有线程组的数量："+Thread.currentThread().getThreadGroup().activeGroupCount());
			
			ThreadGroup[] tgrup=new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];
			Thread.currentThread().getThreadGroup().enumerate(tgrup);
			for(int i=0;i<tgrup.length;i++){
				System.out.println("第一个线程组名称为："+tgrup[i].getName());
			}
		}
方法activeGroupCount()作用是获取当前线程对象中的子线程组的数量
在实例化一个ThreadGroup线程数组X时如果不指定所属线程组，**则x线程组自动归到当前线程所属的线程组中**，也就是隐式地在一个线程组中添加了一个子线程组。

### 2.2 获取根线程组与组里加组
1. 获取根线程:
		public static void main(String[] args) {
			System.out.println("线程："+Thread.currentThread().getName()+" 所在线程组名："+Thread.currentThread().getThreadGroup().getName());
			System.out.println("main线程所在线程组的父线程组名是："+Thread.currentThread().getThreadGroup().getParent().getName());
			System.out.println("main线程所在线程组的父线程组的父线程组名是："+Thread.currentThread().getThreadGroup().getParent().getParent().getName());
		}
**JVM的根线程组就是system**，再取其父线程组则报空指针异常。
2. 往线程组中加线程组:
		public static void main(String[] args) {
			System.out.println("线程组名称"+Thread.currentThread().getThreadGroup().getName());
			System.out.println("线程组中活动的线程数量："+Thread.currentThread().getThreadGroup().activeCount());
			System.out.println("线程组中活动的线程组数量--加线程组前："+Thread.currentThread().getThreadGroup().activeGroupCount());
			
			ThreadGroup newgop=new ThreadGroup(Thread.currentThread().getThreadGroup(),"newGroup");
			
			System.out.println("线程组中活动的线程组数量--加线程组后："+Thread.currentThread().getThreadGroup().activeGroupCount());
		    
		}
运行结果:
		线程组名称main
		线程组中活动的线程数量：1
		线程组中活动的线程组数量--加线程组前：0
		线程组中活动的线程组数量--加线程组后：1

### 2.3 批量"停止"线程组内线程
线程类:

		public class Thread7_10 extends Thread{
			//线程组内的线程批量停止
			public Thread7_10(ThreadGroup tg,String name) {
				super(tg,name);
			}
			
			@Override
			public void run() {
				super.run();
				System.out.println("ThreadName"+Thread.currentThread().getName()+" 开始死循环");
				while(!this.isInterrupted()){
				}
				System.out.println("ThreadName"+Thread.currentThread().getName()+" 死循环结束了");
			}
		}
测试类:

		public static void main(String[] args) {
			try{
				ThreadGroup tg=new ThreadGroup("新线程组");
				for(int i=0;i<5;i++){
					Thread7_10 t1=new Thread7_10(tg, " 线程"+(i+1));
					t1.start();
				}
				Thread.sleep(5000);
				tg.interrupt();
				System.out.println("调用了interrupt方法");
			}catch (InterruptedException e) {
				System.out.println("停止了");
				e.printStackTrace();
			}
		}
调用线程组ThreadGroup的interrupt()方法时可以将该线程组所有正在运行的线程批量"停止",并不是真正的停止，只是打上停止标记。

### 2.4 递归与非递归取得组内对象

	public static void main(String[] args) {
		ThreadGroup mainGroup=Thread.currentThread().getThreadGroup();
		ThreadGroup groupA=new ThreadGroup(mainGroup,"A");
		
		Runnable run=new Runnable() {
			@Override
			public void run() {
				try{
					System.out.println("runMethod!");
					Thread.sleep(5000);
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		};
		 
		ThreadGroup  groupB=new ThreadGroup(groupA,"B");
		//分配空间，但是不一定全部使用
		ThreadGroup[]  listGroup1=new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];
		//传入true是递归取得子孙或子孙组
		Thread.currentThread().getThreadGroup().enumerate(listGroup1,true);
		for(int i=0;i<listGroup1.length;i++){
			if(listGroup1[i]!=null){
				System.out.println(listGroup1[i].getName());
			}
		}
		
		ThreadGroup[]  listGroup2=new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];
		//传入false是非递归取得子对象
		Thread.currentThread().getThreadGroup().enumerate(listGroup2,false);
		for(int i=0;i<listGroup2.length;i++){
			if(listGroup2[i]!=null){
				System.out.println(listGroup2[i].getName());
			}
		}
	}
代码:
Thread.currentThread().getThreadGroup().enumerate(listGroup1,true)：
传入true是递归取得**子孙或子孙组**。在代码中是A,B。
Thread.currentThread().getThreadGroup().enumerate(listGroup2,false)：
传入false是非递归取得**子对象**，不会取孙对象。在代码中是A。
运行结果:

	A
	B
	A

---
## 3.使线程具有有序性
正常情况下，线程在运行多个线程之间执行任务的时机是无序的。
可以通过改造代码的方式使它们的运行具有有序性。
线程类:

		public class Thread7_12 extends Thread{
			//使线程具有有序性
			private Object lock;
			private String showChar;
			private int showNumPosition;
			private int printCount=0;           //统计打印了几个字母
			volatile private static int addNumber=1;
			
			public Thread7_12(Object lock,String showChar,int showNumPosition) {
				super();
				this.lock=lock;
				this.showChar=showChar;
				this.showNumPosition=showNumPosition;
			}
			
			@Override
			public void run() {
				super.run();
				try{
					synchronized (lock) {
						while(true){
							if(addNumber%3==showNumPosition){
								System.out.println("ThreadName="+Thread.currentThread().getName()+" runCount="+addNumber+" showChar="+showChar);
								lock.notifyAll();
								addNumber++;
								printCount++;
								if(printCount==3){          //每个对象打印三次后退出,共三组ABC
									break;
								}
							}else{
								lock.wait();
							}
						}
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
测试类:

		public static void main(String[] args) {
			Object lock=new Object();
			Thread7_12 t1=new Thread7_12(lock, "A", 1);
			Thread7_12 t2=new Thread7_12(lock, "B", 2);
			Thread7_12 t3=new Thread7_12(lock, "C", 0);
			t1.start();
			t2.start();
			t3.start();
		}
测试结果:
三组ABC

---
## 4.SimpleDateFormat非线程安全
1. 错误出现:
SimpleDateFormat主要负责日期的转换与格式化，但是在多线程的情况下使用此类容易造成数据转换及处理的不准确性,因为SimpleDateFormat类并不是线程安全的
线程类:
		public class Thread7_13 extends Thread{
			//SimpleDateFormat出现异常的情况
			private SimpleDateFormat sdf;
			private String dateString;
			
			public Thread7_13(SimpleDateFormat sdf,String dateString) {
				super();
				this.sdf=sdf;
				this.dateString=dateString;
			}
			
			@Override
			public void run() {
				super.run();
				try{
					System.out.println(dateString);
					Date dateRef=sdf.parse(dateString);
					String newDateString=sdf.format(dateRef).toString();
					if(!newDateString.equals(dateString)){
						System.out.println("线程"+Thread.currentThread().getName()+"转换出错了,日期字符串="+dateString+",转换后字符串="+newDateString);
					}
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}	
		}
测试类:
		public static void main(String[] args) {
			SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
			String[] dateStringList=new String[]{"2017-10-14","2017-07-11","2017-11-20","2017-05-01","2015-11-14"};
			Thread[] tl=new Thread[dateStringList.length];
			for(int i=0;i<tl.length;i++){
				tl[i]=new Thread7_13(sdf, dateStringList[i]);
			}
			for(int i=0;i<tl.length;i++){
				tl[i].start();
			}
		}
测试结果:
		2017-07-11
		2015-11-14
		2017-11-20
		2017-10-14
		2017-05-01
		线程Thread-0转换出错了,日期字符串=2017-10-14,转换后字符串=2017-11-20
		Exception in thread "Thread-4" Exception in thread "Thread-1" java.lang.NumberFormatException: multiple points
		java.lang.NumberFormatException: multiple points
使用单例的SimpleDateFormat在多线程环境下处理日期，极易出现日期转换错误的情况。
multiple points出错是因为SimpleDateFormat类并不是线程安全的。
2. 解决异常方法1：创建多个SimpleDateFormat对象
线程类:
		public class Thread7_14 extends Thread{
			//解决异常方法1：创建多个SimpleDateFormat对象
			private SimpleDateFormat ref;
			private String dateString;
			
			public Thread7_14(SimpleDateFormat ref,String dateString) {
				super();
				this.ref=ref;
				this.dateString=dateString;
			}
			
			@Override
			public void run() {
				super.run();
				try{
					Date dateRef=Tool7_14.parse("yyyy-MM-dd", dateString);
					String DateString=Tool7_14.formate("yyyy-MM-dd", dateRef);
					if(!DateString.equals(dateString)){
						System.out.println("ThreadName="+this.getName()+"转换出错了， 日期字符串="+dateString+",转换成日期为="+DateString);
					}
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
测试类:
		public static void main(String[] args) {
			SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
			String[] dateStringList=new String[]{"2017-10-14","2017-07-11","2017-11-20","2017-05-01","2015-11-14"};
			Thread[] tl=new Thread[dateStringList.length];
			for(int i=0;i<tl.length;i++){
				tl[i]=new Thread7_14(sdf, dateStringList[i]);
			}
			for(int i=0;i<tl.length;i++){
				tl[i].start();
			}
		}
3. 解决异常方法2： 使用ThreadLocal类
使用ThreadLocal类能使线程绑定到指定对象，使用该类也能解决多线程下SimpleDateFormat的异常。
线程类:
		public class Thread7_15 extends Thread{
			// 解决异常方法2： 使用ThreadLocal类
			private SimpleDateFormat ref;
			private String dateString;
			
			public Thread7_15(SimpleDateFormat ref,String dateString) {
				super();
				this.ref=ref;
				this.dateString=dateString;
			}
			@Override
			public void run() {
				super.run();
				try{
					Date dateRef=Tool7_15.getsimpledateformat("yyyy-MM-dd").parse(dateString);
					String DateString=Tool7_15.getsimpledateformat("yyyy-MM-dd").format(dateRef).toString();
					if(!DateString.equals(dateString)){
						System.out.println("ThreadName="+this.getName()+"转换出错了， 日期字符串="+dateString+",转换成日期为="+DateString);
					}	
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
测试类:
		public static void main(String[] args) {
			SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
			String[] dateStringList=new String[]{"2017-10-14","2017-07-11","2017-11-20","2017-05-01","2015-11-14"};
			Thread[] tl=new Thread[dateStringList.length];
			for(int i=0;i<tl.length;i++){
				tl[i]=new Thread7_15(sdf, dateStringList[i]);
				
			}
			for(int i=0;i<tl.length;i++){
				tl[i].start();
			}
		}
同一个线程再次使用不用创建新的SimpleDateFormat对象，比上一个方法效率更高。

---
## 5.线程组异常的处理
1. 线程中出现的异常：
thread.setUncaughtExceptionHandler(UncaughtExceptionHandler)方法:
**对【指定的线程对象】设置默认的异常处理器。**
Thread.setDefaultExceptionHandler(UncaughtExceptionHandler)方法:
**为【指定线程类的所有线程对象】设置默认的线程处理器。**
对类与对象一起添加默认的处理，遇到时只执行对象的。
线程类:
		public class Thread7_16 extends Thread{
			//线程中出现异常的处理
			@Override
			public void run() {
				super.run();
				String username=null;
				System.out.println(username.hashCode());
			}
		}
测试类:
		public static void main(String[] args) {
			Thread7_16 t1=new Thread7_16();
			t1.setName("线程1");
			UncaughtExceptionHandler eh=new UncaughtExceptionHandler() {
				@Override
				public void uncaughtException(Thread t, Throwable e) {
					System.out.println("线程："+t.getName()+"   出现异常：");
					e.printStackTrace();
				}
			};
			t1.setUncaughtExceptionHandler(eh);
			//Thread7_16.setDefaultUncaughtExceptionHandler(eh);
			
			Thread7_16 t1=new Thread7_16();
			t1.setName("线程t1");
			t1.start();
			Thread7_16 t2=new Thread7_16();
			t2.start();
			t2.start();		
		}
2. 线程组内异常的处理:
一个线程出现异常，线程组内其他线程会不会停止
线程组一个线程出现异常不会影响其他线程的运行
想要实现一个线程异常，组内其他线程也停止运行:
使用自定义线程组类，并重写uncaughtException()方法
自定义线程组类:
		public class ThreadGroup7_17_2 extends ThreadGroup{
			//线程组内处理异常
			public ThreadGroup7_17_2(String name) {
				super(name);
			}
			
			//方法uncaughtException(Thread t, Throwable e)中的t是出现异常的线程对象
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				super.uncaughtException(t, e);
				this.interrupt();
			}
		}
线程类:
		public class Test7_17_2 {
			//线程组内处理异常
			public static void main(String[] args) {
				ThreadGroup group=new ThreadGroup("线程组");
				Thread7_17_2[] tlist=new Thread7_17_2[10];
				for(int i=0;i<tlist.length;i++){
					tlist[i]=new Thread7_17_2(group, "线程"+(i+1),"1");
					tlist[i].start();
				}
				Thread7_17_2 th=new Thread7_17_2(group, "报错线程", "a");
				th.start();
			}
		}
测试类:
		public static void main(String[] args) {
			ThreadGroup7_17_2 group=new ThreadGroup7_17_2("线程组");
			Thread7_17_2[] tlist=new Thread7_17_2[10];
			for(int i=0;i<tlist.length;i++){
				tlist[i]=new Thread7_17_2(group, "线程"+(i+1),"1");
				tlist[i].start();
			}
			Thread7_17_2 th=new Thread7_17_2(group, "报错线程", "a");
			th.start();
		}
注意:使用自定义ThreadGroup线程组并且重写uncaughtException方法处理组内线程中断行为时，**每个线程对象内都不能有catch语句**，如果有，则uncaughtException(Thread t, Throwable e)方法不执行。
3. 线程异常处理的传递:
对象异常处理类:
		public class ObjectUncaughtExceptionHandler implements UncaughtExceptionHandler{
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				System.out.println("对象的异常处理");
				e.printStackTrace();
			}
		}
静态的异常处理类:
		public class StateUncaughtExceptionHandler implements UncaughtExceptionHandler{
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				//uncaughtException(t, e);
				System.out.println("静态的异常处理");
				e.printStackTrace();
			}
		}
自定义线程组类（线程组异常处理）:
		public class ThreadGroup7_18 extends ThreadGroup{
			//线程异常处理的传递
			public ThreadGroup7_18(String name) {
				super(name);
			}
			
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				super.uncaughtException(t, e);
				System.out.println("线程组的异常处理");
				e.printStackTrace();
			}
		}
测试类:
		public static void main(String[] args) {
			ThreadGroup7_18 tg=new ThreadGroup7_18("wod线程组");
			Thread7_18 th=new Thread7_18(tg,"线程1");
			
			//对象
			//th.setUncaughtExceptionHandler(new ObjectUncaughtExceptionHandler());  //注释掉这一行
			
			//类
			Thread7_18.setDefaultUncaughtExceptionHandler(new StateUncaughtExceptionHandler());
			
			th.start();
		}

**有对象异常处理的执行对象的；
没有对象异常处理的执行类，静态和线程组的异常处理。**
修改测试类:
	public static void main(String[] args) {
		ThreadGroup7_18 tg=new ThreadGroup7_18("wod线程组");
		Thread7_18 th=new Thread7_18(tg,"线程1");
		th.start();
	}
则静态的线程处理不会打印`System.out.println("静态的异常处理");`
想要打印出静态处理的信息必须在 public void uncaughtException(Thread t, Throwable e)中加上uncautException(t,e);
注意将静态异常处理作为传到类异常处理时不能加uncautException(t,e)，否则会出现栈溢出错误。

---
