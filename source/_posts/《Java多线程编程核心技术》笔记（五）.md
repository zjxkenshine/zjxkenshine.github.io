---
title: 《Java多线程编程核心技术》笔记（五）:定时器Timer
date: 2018-03-05 11:03:11
tags:
- Java
- 多线程
categories: 学习笔记

---
## 0.学习要点
- 如何实现在指定时间执行任务
- 如何实现按指定周期执行任务
- schedule(）方法和scheduleAtFixedRate()方法

---
## 1.Timer简介
定时/计划功能在移动开发用的较多，如Android。定时计划任务功能在Java中主要使用的就是Timer对象，它在内部是使用多线程方式进行处理的。
**Timer类**的主要是负责计划任务的功能，就是*在指定时间开始执行某一任务。*
**Timer类**主要作用是设置计划任务，但是封装任务的类却是**TimerTask类。**
执行计划任务的代码要放入TimerTask的子类中，因为**TimerTask是一个抽象类**。

---
## 2.schedule(TimerTask task,Date time)
1. 方法作用:
*在指定时间time执行一次某任务task。*
简单使用:
		private static Timer time=new Timer();
		static public class MyTask extends TimerTask{
			@Override
			public void run() {
				//任务代码
			}
		}
		public static void main(String args[]){
			MyTask tsk=new MyTask();
			Date dt=...;
			time.schedule(tsk, dt);  //执行任务
		}
2. 执行任务时间晚于当前时间:在未来执行的效果
		public class Test5_02_1 {
			private static Timer time=new Timer();
			
			static public class MyTask extends TimerTask{
				@Override
				public void run() {
					System.out.println("运行了,时间为"+System.currentTimeMillis());
				}
			}
			
			public static void main(String[] args) {
				try{
					MyTask tsk=new MyTask();
					SimpleDateFormat sdf=new SimpleDateFormat("yy-MM-dd HH:mm:ss");
					String dateString="2018-03-05 11:57:00";
					Date dt=sdf.parse(dateString);
					System.out.println("字符串时间："+dt.toLocaleString()+"当前时间："+new Date().toLocaleString());
					time.schedule(tsk, dt);
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
运行后程序在指定的时间运行了，但是程序并没有销毁。因为Timer创造的线程仍在运行，并未停止。将它改为守护线程即可，创建时这样写:
		private static Timer time=new Timer(true);
3. 计划时间早于当前时间：提前运行的效果。
将上述代码中的```String dateString="2018-03-05 11:57:00";```改为比当前时间早的时间，测试运行的到结果:
**如果执行任务时间早于当前时间，则立即执行task任务。**
4. 多个TimerTask延时测试:
		public class Test5_04_2 {	
			private static Timer timer=new Timer();  //不能使用守护线程
			
			static public class Tsk1 extends TimerTask{
				@Override
				public void run() {
					try {
					    System.out.println("tsk1 begin运行了,时间为"+System.currentTimeMillis());
						Thread.sleep(10000);
						System.out.println("tsk1 end运行了,时间为"+System.currentTimeMillis());
						
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			
			static public class Tsk2 extends TimerTask{
				@Override
				public void run() {
					System.out.println("tsk2运行了,时间为"+System.currentTimeMillis());
				}
			}
			
			public static void main(String[] args) {
				try{
					Tsk1 tsk1=new Tsk1();
					Tsk2 tsk2=new Tsk2();
					SimpleDateFormat sdf1=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
					SimpleDateFormat sdf2=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
					String day1="2017-10-10 21:56:00";
					String day2="2017-10-10 21:56:00";
					
					Date d1=sdf1.parse(day1);
					Date d2=sdf2.parse(day2);
					
					System.out.println("字符串1计划执行时间："+d1.toLocaleString()+" 当前实际执行的时间："+new Date().toLocaleString());
					System.out.println("字符串2计划执行时间："+d1.toLocaleString()+" 当前实际执行的时间："+new Date().toLocaleString());
					
					timer.schedule(tsk1, d1);
					timer.schedule(tsk2, d2);
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
运行结果:
		字符串1计划执行时间：2017-10-10 21:56:00 当前实际执行的时间：2018-3-5 12:00:12
		字符串2计划执行时间：2017-10-10 21:56:00 当前实际执行的时间：2018-3-5 12:00:12
		tsk1 begin运行了,时间为1520222412009
		tsk1 end运行了,时间为1520222422014
		tsk2运行了,时间为1520222422014
结论:
TimerTask是以队列的方式**一个一个被顺序执行**，所以执行的时间有可能和预计的时间不一样前面的任务耗时可能较长，则后面的任务运行的时间也会被延迟。

---
## 3.schedule(TimeTask task,Date firstTime,long period)方法
1. 作用:
*在指定日期（firstTime）之后，按指定的间隔（period）周期性地无限循环执行某一任务（task）。*
简单使用:
		private static Timer time=new Timer();
		static public class MyTask extends TimerTask{
			@Override
			public void run() {
				//任务代码
			}
		}
		public static void main(String args[]){
			MyTask tsk=new MyTask();
			Date firstTime=...;
			time.schedule(tsk,firstTime，4000);  //执行任务，4秒一循环
		}
2. 计划时间晚于当前时间:在未来执行的效果。
从firstTime开始，无限循环执行该任务。
3. 计划时间早于当前时间:提前运行的效果。
**程序立即执行，并以period为周期无限循环执行该任务。**
4. 任务执行时间被延时。
		public class Test5_07 {
			static public class MyTask extends TimerTask{
				@Override
				public void run() {
					try{
						System.out.println("A运行了，时间为"+new Date());
						Thread.sleep(5000);
						System.out.println("A运行结束了，时间为"+new Date());
					}catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			
			public static void main(String[] args) {
				try{
					MyTask tsk=new MyTask();
					SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
					String dString="2018-3-5 12:45:30";
					Timer time=new Timer();
					Date d1=sdf.parse(dString);
					System.out.println("执行时间"+d1.toLocaleString()+"现在时间"+new Date().toLocaleString());
					time.schedule(tsk, d1, 1000);              //1秒循环一次
					//会延迟为5秒循环一次
				}catch(ParseException e){
					e.printStackTrace();
				}
			}
		}
原本1秒执行一次任务，因为延迟执行，变为5秒执行一次。

---
## 4.TimerTask与Timer类的cancel()方法
1. TimerTask的cancel()方法：
*作用是将自身从任务队列中清除。*
	static public class MyTsk1 extends TimerTask{
		@Override
		public void run() {
			System.out.println("任务1执行了，时间"+new Date());
			this.cancel();
		}
	}
	
	static public class MyTsk2 extends TimerTask{
		@Override
		public void run() {
			System.out.println("任务2执行了，时间"+new Date());
		}
	}
若同时实例化两个对象并且用Timer对象的方法(schedule)去启动这两个task对象，则MyTsk1会在输出一次后停止。而MyTsk2的对象将会继续执行。
2. Timer类的cancel()方法:
*将任务队列(timer)中的全部任务清空。*
		private static Timer timer=new Timer();
		
		static public class MyTsk1 extends TimerTask{
			@Override
			public void run() {
				System.out.println("线程1运行了，时间为："+new Date());
				timer.cancel();
			}
		}
		
		static public class MyTsk2 extends TimerTask{
			@Override
			public void run() {
				System.out.println("线程2运行了，时间为："+new Date());
			}
		}
若此时使用timer启动这两个task对象，则两个任务都会停止。
若在启动这两个对象前重新实例化Timer timer=new Timer();
则这两个任务都不会停止。
3. 使用timer.cancel()的意外:
		public class Test5_10 {
			//Timer类中的cancel()方法的注意事项
			static int i=0;
			static public class Mtsk extends TimerTask{
				@Override
				public void run() {
					System.out.println("正常执行了"+i);
				}
			}
			
			public static void main(String[] args) {
				while(true){
					try{
						i++;
						Timer timer=new Timer();
						Mtsk tsk=new Mtsk();
						SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
						String dString="2017-10-11 19:20:00";
						Date d1=sdf.parse(dString);
						timer.schedule(tsk, d1);  //有时它抢到锁
						timer.cancel();  //有时它抢到锁
					}catch (ParseException e) {
						e.printStackTrace();
					}
				}
			}
		}
出现意外的原因是因为Tiemr类中的cancel()方法有时并没有争抢到queue锁，TimerTask类的任务正常执行。

---
## 5.schedule(TimerTask task,long delay)和schedule(TimerTask task,long delay，long period)
1. schedule(TimerTask task,long delay)方法：
*以执行schedule(TimerTask task,long delay)方法当前为参考时间，在此基础上延迟指定毫秒数（delay）后执行一次task任务。*
		static public class MyTask extends TimerTask{
			@Override
			public void run() {
				System.out.println("运行了,时间为:"+new Date());
			}
		}
		public static void main(String[] args) {
			MyTask tsk=new MyTask();
			Timer time=new Timer();
			System.out.println("当前时间："+new Date().toLocaleString());
			time.schedule(tsk, 5000);
		}
2. schedule(TimerTask task,long delay，long period)方法:
*从执行方法后的delay毫秒后开始以period毫秒为周期无限次得循环执行task任务*
	static public class MyTsk extends TimerTask{
		@Override
		public void run() {
			System.out.println("运行了，时间为："+new Date());
		}
	}
	public static void main(String[] args) {
		MyTsk tsk=new MyTsk();
		Timer time=new Timer();
		System.out.println("当前时间："+new Date().toLocaleString());
		time.schedule(tsk, 5000,1000);    //5秒后以一秒为周期无限循环执行
	}

---
## 6.scheduleAtFixedRate(TimerTask task,Date firstTime,long period)方法
1. **方法scheduleAtFixedRate和schedule的区别**:
a.方法scheduleAtFixedRate和schedule都会按顺序执行，不会出现非线程安全。
b.方法scheduleAtFixedRate和schedule的区别只体现在不延时的情况：
schedule:*若没延时，下一个任务的执行时间参考的是**上一次任务“开始”的时间**来计算*
scheduleAtFixedRate：*无延迟时scheduleAtFixedRate是以**上一个任务结束时间**为参考作为开始时间的（若task中含sleep则下一个开始时间就是上一个结束时间）。*
c.延时的情况下没有区别：若被延时，下一个任务的执行时间都是上一次任务“结束”的时间来计算
d.schedule不具有任务追赶性,scheduleAtFixedRate具有任务追赶性
2. schedule方法任务不延时
		public class Test5_14 {
			//测试schedule方法任务不延时
			private  static Timer timer=new Timer(false);
			private static int runCount=0;
			
			static public class MyStack extends TimerTask{
				@Override
				public void run() {
					try {
					System.out.println("任务开始执行，时间为"+new Date());
					Thread.sleep(1000);
					System.out.println("任务执行结束，时间为"+new Date());
					runCount++;
					if(runCount==5){
						timer.cancel();
					}
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			public static void main(String[] args) {
				try{
				MyStack mtsk=new MyStack();
				SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
				String start="2017-10-11 20:29:00";
				Date day=sdf.parse(start);
				System.out.println("起始时间为"+day.toLocaleString()+"当前时间"+new Date().toLocaleString());
				timer.schedule(mtsk, day, 3000);          //间隔为3秒，线程运行1秒      
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
在不延时的情况下，schedule下一次执行任务的时间是上一次任务开始的时间加上delay（period）
3. scheduleAtFixedRate(TimerTask task,Date firstTime,long period)任务不延时：
		public class Test5_16 {
			/**对比Test5_14
			 * 在不延时的情况下，scheduleAtFixedRate下一次执行任务的时间是上一次任务结束的时间
			 */
			private static Timer timer=new Timer();
			private static int runCount=0;
			
			static public class MyStack extends TimerTask{
				@Override
				public void run() {
					try {
					System.out.println("任务开始执行，时间为"+new Date());
					Thread.sleep(1000);
					System.out.println("任务执行结束，时间为"+new Date());
					runCount++;
					if(runCount==5){
						timer.cancel();
					}
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			public static void main(String[] args) {
				try{
				MyStack mtsk=new MyStack();
				SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
				String start="2017-10-11 20:29:00";
				Date day=sdf.parse(start);
				System.out.println("起始时间为"+day.toLocaleString()+"当前时间"+new Date().toLocaleString());	
				timer.scheduleAtFixedRate(mtsk,day,3000);          //scheduleAtFixedRate间隔为3秒，线程运行2秒      	
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}
运行结果:
		起始时间为2017-10-11 20:29:00当前时间2018-3-5 23:00:07
		任务开始执行，时间为Mon Mar 05 23:00:07 CST 2018
		任务执行结束，时间为Mon Mar 05 23:00:08 CST 2018
		任务开始执行，时间为Mon Mar 05 23:00:08 CST 2018
		任务执行结束，时间为Mon Mar 05 23:00:09 CST 2018
		任务开始执行，时间为Mon Mar 05 23:00:09 CST 2018
		任务执行结束，时间为Mon Mar 05 23:00:10 CST 2018
		任务开始执行，时间为Mon Mar 05 23:00:10 CST 2018
		任务执行结束，时间为Mon Mar 05 23:00:11 CST 2018
		任务开始执行，时间为Mon Mar 05 23:00:11 CST 2018
		任务执行结束，时间为Mon Mar 05 23:00:12 CST 2018
结论:无延迟时scheduleAtFixedRate是以上一个任务结束时间为参考作为开始时间的（若task中含sleep则下一个开始时间就是上一个结束时间）。
4. scheduleAtFixedRate与schedule延时：
与无延迟时scheduleAtFixedRate方法(task中含sleep)一样，是以上一个任务结束时间作为开始时间的。
5. 任务追赶性:
开始时间早于当前时间时，这中间的任务就忽略执行了，这就是schedule不具有任务追赶性，而scheduleAtFixedRate会把这段时间未执行的任务追赶回来：
		public class Test5_19 {
			private static Timer timer=new Timer();
			
			static public class MyTask extends TimerTask{
				@Override
				public void run() {
					System.out.println("1 begin 运行了，时间为"+new Date());
					System.out.println("1 end   运行了，时间为"+new Date());
				}
			}
			public static void main(String[] args) {
				try{
					MyTask stk=new MyTask();
					SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
					String dString ="2018-03-05 23:11:00";  //早于当前时间
					Date day=sdf.parse(dString);
					System.out.println("开始时间"+day.toLocaleString()+"当前时间"+new Date().toLocaleString());
					
					timer.scheduleAtFixedRate(stk, day, 5000);
				}catch (ParseException e) {
					e.printStackTrace();
				}
			}
		}


---