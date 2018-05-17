---
title: JVM学习笔记（五）：性能监控工具与Java堆分析
date: 2018-5-15 17:17:17 
tags: 
- JVM
- Linux
categories: Java基础

---
## 0.学习准备
1. 参考资料
参考书籍《深入理解Java虚拟机》
参考视频《深入理解JVM》(目前学习)
2. 简单目录：
	- 系统性能监控
	- Java自带的性能监控工具
	- 内存溢出OOM的原因
	- MAT使用基础
		- 浅堆与深堆
		- 显示入引用与出引用
		- 支配树
	- 使用JVisualVM分析堆

---
## 1.系统监控命令(Linux系统)
1. uptime命令：查询系统时间
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-02.jpg)
		19:49:25(当前时间) up 9 min(系统运行时间),  4 users(连接数),  load average: 0.01, 1.00, 0.91(平均负载)
	- 连接数：
		- 不是指有多少个用户，而是有多少个终端连接该服务
		- 一个终端算一个连接
	- 平均负载：
		- 分别是1，5，15分钟内的平均负载
		- 通过运行队列中的平均进程数表示
		- 值越大表示负载越重
2. top命令：统计cpu详细信息
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-01-1.jpg)
1.第一行的输出和uptime一样
2.后面可以看见CPU内存和交换空间的使用情况：
		Tasks: 171 total,   2 running, 169 sleeping,   0 stopped,   0 zombie
		%Cpu(s):  6.8 us,  9.5 sy,  0.0 ni, 78.2 id,  5.1 wa,  0.0 hi,  0.3 si,  0.0 st
		KiB Mem :  1016232 total,    70848 free,   729436 used,   215948 buff/cache
		KiB Swap:  1679356 total,  1569956 free,   109400 used.   104608 avail Mem 
如果swap空间大量使用，说明系统的实际运行内存有所欠缺。会引起大量的I/O，印象系统性能。(类似页式存储内存和硬盘交换)
3.后面的则是每个进程占cpu的使用情况
		PID(线程id)	USER(用户)	PR(调度优先级)	NI(进程优先级)	VIRT(虚拟内存)	RES(常驻内存)	SHR(共享内存)	S(进程状态)	%CPU %MEM	TIME+ COMMAND
	>S：这个是进程的状态。它有以下不同的值:
	- D - 不可中断的睡眠态。
	- R – 运行态
	- S – 睡眠态
	- T – 被跟踪或已停止
	- Z – 僵尸态
	
	>%CPU：自从上一次更新时到现在任务所使用的CPU时间百分比。
	%MEM：进程使用的可用物理内存百分比。
	TIME+：任务启动后到现在所使用的全部CPU时间，精确到百分之一秒。
	COMMAND：运行进程所使用的命令。进程名称（命令名/命令行）
3. vmstat命令：统计cpu信息
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-03.jpg)
	- 统计系统系统cpu,内存，swap,io等情况
	- CPU占用高，上下文切换频繁，说明系统有线程正在频繁切换
	- 详细的字段说明可以查看：<http://man.linuxde.net/vmstat>
4. pidstat命令：细致观察进程
一般系统不自带，需要安装：（我的Linux已经安装了）
		sudo apt-get install sysstat(ubantu中)
可以监控CPU,IO以及内存等：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-04.jpg)
可以使用以下参数：
		-p：指定进程pid
		-u：指定查询哪个cpu
		-t：显示线程
		-d：显示磁盘I/O的情况
如果在Linux中安装了Java则可以通过-t查询哪个java线程占用CPU过高，然后对该进程做一些优化。(cpu瓶颈)
I/O分为磁盘I/O和网络I/O，使用-d参数可以查看那些线程占用的IO较高(IO瓶颈)。

---
## 2.系统监控（Windows）
**1)图形化监控工具：**
1. 任务管理器：
【Ctrl】+【Alt】+【Delete】
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-05.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-06.jpg)
右键可以设置显示哪些信息。
可以在任务管理器中查看cpu占用率，磁盘IO，网络IO等
2. Perfmon：Windows自带的多功能监控工具
功能比任务管理器丰富(功能强大，但是使用起来不太方便)
具体的使用方法查看博客：
<https://blog.csdn.net/oscar999/article/details/7918385>
简单的使用：【Windows】+【R】输入perfmon点击确定即可
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-07.jpg)
打开之后界面如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-08.jpg)
运行java应用之后再该界面右键选择Java进程就可以进行相关的监控。

**2)命令监控工具：**
1. 图形化工具好用，但是很难进行脚本化处理，很难做批量处理，如果需要定时定点获取cpu使用情况或者自动化测试等，那么使用命令行工具更为合适。
2. pslist命令：
默认是没有这个命令的
微软官网下载PSTools压缩包，解压后将pslist.exe复制到C:\Windows\System32目录下，双击运行，再到cmd界面使用命令pslist查看所有进程信息。
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-09.jpg)
使用pslist进程名或pslist -dmx pid号查看具体某个进程的具体信息。

---
## 3.Java自带的工具
通常会使用系统监控工具初步定为是不是Java程序引起的。
然后再通过Java自带的工具来具体分析。
在安装路径的\bin目录下可以找到这些可执行文件。
在tools.jar中可以找到这些工具类：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-10.jpg)

### jps,jinfo,jmap,jstack
**1.jps命令：**
1. 作用：
列出java进程，类似于ps命令
2. 参数：
	- -q：指定jps只输出进程ID。不会显示类的信息。
	- -m：可以输出传递给Java进程(主函数)的参数。
	- -l：可以输出主函数的完整路径。
	- -v：可显示传递给JVM的参数。
3. 使用jps命令时出现的问题：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-11.jpg)
使用管理员方式打开cmd就可以使用了。
4. 测试
测试代码：(一直使用这个)
		public class jpsTest {
			public static void main(String[] args) {
				Timer timer = new Timer();
				timer.schedule(new TimerTask() {
					@Override            
					public void run() {
						System.out.println(Thread.currentThread() + " is running");
					}
				}, new Date(), 6000);
			}
		}
运行代码：
		Thread[Timer-0,5,main] is running
		Thread[Timer-0,5,main] is running
		Thread[Timer-0,5,main] is running
		Thread[Timer-0,5,main] is running
		Thread[Timer-0,5,main] is running
		Thread[Timer-0,5,main] is running
		...
在命令行使用jps查询：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-13.jpg)
-m与-v参数测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-12.jpg)

**2)jinfo命令：**
1. 可以用来查看当前正在运行的Java程序的扩展参数。
甚至可以在运行时修改部分参数。
2. 用法：
	- `jinfo -flags <name>`：打印指定JVM的参数值
	- `jinfo -flags [+|-]<name>`：设置指定JVM的布尔值
	- `jinfo -flags <name>=<value>`：设置指定JVM的参数值
4. 使用测试：
测试命令：(获取该pid的JVM参数信息)
		jinfo -flags 10740 //使用jps获取的pid
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-14.jpg)

**3)jmap命令：**
1. `jmap histo pid [>文件]`
生成Java程序的堆快照和对象的统计信息
2. 简单使用：
`jmap -histo 8580`
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-15.jpg)
后面还有非常多的信息，可以使用数据流重定向`>`将信息保存在某一文件中，如：
`jmap -histo 8580 >c:/log.txt`
3. 备份堆信息
`jmap -dump:live,format=b,file=D:/dump.hprof 8580`
4. 其他的使用方式可以使用`jmap -help`查询。
命令格式：
		jmap [ option ] pid
		jmap [ option ] executable core
		jmap [ option ] [server-id@]remote-hostname-or-IP
参数：
		option：选项参数，不可同时使用多个选项参数
		pid：java进程id，命令ps -ef | grep java获取
		executable：产生核心dump的java可执行文件
		core：需要打印配置信息的核心文件
		remote-hostname-or-ip：远程调试的主机名或ip
		server-id：可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器
options参数：
		heap : 显示Java堆详细信息
		histo : 显示堆中对象的统计信息
		permstat :Java堆内存的永久保存区域的类加载器的统计信息
		finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
		dump : 生成堆转储快照
		F : 当-dump没有响应时，强制生成dump快照

**4)jstack：**
1. 作用：
打印线程的dump
2. 参数:
不加参数只打印基本信息。
`-l`：打印锁信息
`-m`：打印java和native的帧信息
`-F`：强制备份，当jstack没有响应时使用
3. 测试：
`jstack -l 8580 //pid`
测试结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-16.jpg)
有很多信息，往下翻还可以看见一些GC线程信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-17.jpg)
其中tid是线程在java程序中的id,nid(native id)是线程在操作系统中的id。

### 图形化工具：JConsole，JVisualVM
**1)JConsole**
1. 图形化监控工具，有如下作用：
	- 查看运行概况
	- 监控堆信息
	- 监控永久区(1.8后则是元数据空间)
	- 监控类加载情况
2. 使用：
在命令行输入`jconsole`(无论大小写)，然后出现这样的界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-18.jpg)
选择想要监控的进程点击连接即可，然后可以进行监控：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-19.jpg)
内存监控：(监控各个区的使用情况)
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-20.jpg)
右上角有个执行GC,点击之后可以强制执行一次GC:
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-21.jpg)
线程监控，下方可以检测死锁：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-22.jpg)
其余还有很多监测功能，不再具体介绍了。
3. 其他具体的信息分析以后看书再补充。

**2)JVisualVM：**
1. JVisualVM是一个多合一的故障诊断和性能监控的可视化工具。
基本上JConsole能做的它也能做，还能做许多JConsole不能做的。
还可以做性能分析等。（all-in-one）
2. 使用：
在命令行使用JVisualVM就可以打开监控页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-23.jpg)
在左侧选择需要监控的进程，然后连接之后就可以对该进程进行监控：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-24.jpg)
线程监控及死锁检查：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-25.jpg)
3. profile功能(性能分析)
可以对cpu，内存进行监测(查询占用cpu较高的应用并优化)
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-26.jpg)
4. 分析Dump堆的功能：
出现OOM时一般会将堆内存Dump出来到一个文件，JVisualVM就是一个可以打开Dump堆并分析的工具。(点击左上角装入备份文件即可)
具体的操作以后再补充。

---
## 4.内存溢出(OOM)的原因
**1)OOM简介：**
1. JVM会引起OOM的内存区间：
	- 堆
	- 永久区
	- 线程栈
	- 直接内存
2. 堆和永久区的内存溢出前面已经有过简单学习了：
[JVM学习笔记（二）：常用参数与类加载器](https://zjxkenshine.github.io/2018/05/07/JVM%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9A%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0%E4%B8%8E%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8/)

**2)堆溢出：**
1. 在内存溢出中最常见的一个问题：
程序过度使用堆空间，没有及时释放一些无用的对象
2. 测试：
测试代码：
		public class HeapOOMTest {
			public static void main(String[] args) {
				ArrayList<byte[]> list=new ArrayList<byte[]>();
				while(true){
					list.add(new byte[1024*1024]);
				}
			}
		}
运行结果:
		Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
			at chap07.HeapOOMTest.main(HeapOOMTest.java:9)
因为在List中的对象不会被认为是垃圾对象，所以一直不会释放，一直往里面添加肯定会溢出。
3. 解决HeapOOM的方法：
	- 增大最大堆空间
	- 及时释放内存

**3)永久区溢出：**
1. 永久区的类过多溢出：
如使用Cglib生成很多个代理类：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-14.jpg)
不过方法区和“PermGen space”又有着本质的区别。前者是 JVM 的规范，而后者则是JVM规范的一种实现，并且只有 HotSpot 才有 “PermGen space”
2. 只有jdk7之前才有永久代的说法，java8已经取消了永久区使用元数据空间。
永久区和元数据空间的区别可以查看博客：
[永久代（PermGen）和元空间的区别（Metaspace）](https://blog.csdn.net/u011531613/article/details/62971713)
3. 异常：
		java.lang.OutOfMemoryError: PermGen space
4. 解决方法：
	- 增大永久区
	- 允许回收Class文件

**4）栈内存溢出：**
1. 栈溢出指创建线程的时候要为线程分配栈空间，这个栈空间是操作系统请求的，如果栈空间给不出足够的空间就会抛出OOM。
2. 堆空间+栈空间之和不能超出操作系统分配给JVM的总空间。
3. 测试栈溢出：(需要使用32位虚拟机运行)
测试代码：
		public class TaskOOMThread implements Runnable{
			@Override
			public void run() {
				try{
					Thread.sleep(1000000);
				}catch(Exception e){
					e.printStackTrace();
				}
			}
			
		}
		public class StackOOMTest {
			public static void main(String[] args) {
				for(int i=0;i<100000;i++){
					new Thread(new TaskOOMThread(),"Thread"+i).start();
					System.out.println("已创建线程"+i);
				}
			}
		}
测试参数：(如果操作系统内存较大则建议调大点)
`-Xmx1g -Xss2m`
正常情况下的运行结果：
		java.lang.OutofMemoryError: unable to create new native thread
4. 注意在单线程中使用递归无限循环时报错都是：
		java.lang.StackOverflowError
不会报OOM异常。
5. 如何解决栈溢出异常：
	- 减少堆内存
	- 减少线程栈大小
6. 64位JDK可能会一般不会出现栈或直接内存溢出：
可以使用`java -d32`或者`java -d64`查看是几位的,不是该版本就会报错。

**5)直接内存溢出：**
1. 和栈溢出类似。
2. 堆空间+栈空间+直接内存使用空间之和不能大于操作系统分配的可用空间。
3. 测试：(需要使用32位虚拟机运行)
测试代码：
		public class DirectOOMTest {
			public static void main(String[] args) {
				int i=0;
				while(true){
					i++;
					System.out.println(i);
					ByteBuffer.allocateDirect(1024*1024);
				//	System.gc();
				}
			}
		}
测试参数：
		-Xmx1g -XX:+PrintGCDetails
测试结果：
		732  
		733  
		Exception in thread "main" java.lang.OutOfMemoryError  
			at sun.misc.Unsafe.allocateMemory(Native Method)  
			at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:127)  
			at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:306)  
			at geym.zbase.ch7.oom.DirectBufferOOM.main(DirectBufferOOM.java:14)
具体可以参照：
<http://book.51cto.com/art/201504/472203.htm>
4. 解决直接内存溢出：
	- 减小堆内存
	- 手动触发GC(上面代码的注释去掉)

---
## 5.MAT基本使用
**1)MAT简介：**
1. Memory Analyzer Tool（MAT）,内存分析工具
是一款基于Eclipse的插件
2. 下载安装：
【Help】-->【Install New Software】,输入`http://download.eclipse.org/mat/1.6/update-site/`：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-27.jpg)
全选之后一路点击Next进行安装,然后安装完毕后点击Finish，重启Eclipse就安装完毕了。
3. 也可以单独下载安装MAT工具，地址为：
<http://www.eclipse.org/mat/downloads.php>

**2)使用MAT打开heap dump：**
1. 使用方式一，jmap+MAT：
运行代码：
		public class jpsTest {
			public static void main(String[] args) {
				Timer timer = new Timer();
				timer.schedule(new TimerTask() {
					@Override            
					public void run() {
						System.out.println(Thread.currentThread() + " is running");
					}
				}, new Date(), 6000);
			}
		}
使用jps查看进程：
		C:\Windows\system32>jps
		94468 jpsTest
		95956 Jps
		99268
使用jmap备份堆信息：
		jmap -dump:live,format=b,file=D:/Dump/jpsTest.hprof 94468
备份结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-29.jpg)
在Eclipse左上角选择Memory Analyze视图：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-30.jpg)
在左上角选择File-->Open Heap Dump打开备份文件即可：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-28-1.jpg)
2. 分析溢出的堆内存：
测试代码：
和上面堆内存溢出的测试代码相同。
测试参数：
		-XX:+HeapDumpOutOnOfMemoryError -XX:HeapDumpPath=D:/Dump
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-31.jpg)
然后就可以和上面一样进行分析。
3. 在使用MAT打开.hprof文件之后，当前目录下会出现许多文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-32.jpg)
4. 最后的压缩文件，解压查看，其中的html文件保存的就是刚刚的分析结果，
底部可以查看可能出现内存泄露的地方：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-33.jpg)

**3)MAT的简单使用：**
1. MAT的界面：(可以先查看一波后面的MAT分析相关知识)
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-34.jpg)
从左往右分别是：
总览，柱状图，支配树，对象查询语言（OQL）,线程对象信息，堆的信息列表，对象消耗情况列表，搜索
2. 堆的信息列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-35.jpg)
3. 对象信息列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-36.jpg)
4. 在类上右键则可以查看该类对象的入引用和出引用：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-37.jpg)
5. 在线程对象信息中可以查看到浅堆和深堆的信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-38.jpg)
6. 更加精细的分析以后再去深究。

---
## 6.MAT分析相关知识
**1)对象引用图及支配树：**
1. 对象引用图及支配树：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-39.jpg)
2. 支配：
如果在对象引用图中所有的通往B的路径都经过A，则称A是B的支配者。
3. 直接支配者：
A是离B最近的支配者，则称A是B的直接支配者。
将直接支配者连线则可以构成支配树。
4. 支配者被回收，被支配的所有对象也会被回收。
5. 支配者视图可以很简单的得知一个对象被回收会关联回收多少空间：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-40.jpg)

**2)出引用和入引用：**
1. 出引用：
该类对象被哪些类所引用
2. 入引用：
该类引用了哪些类

**3)浅堆和深堆：**
1. 浅堆：
*一个对象结构所占用的的内存大小*
	- 关于对象的大小可以查看：<https://blog.csdn.net/zqz_zqz/article/details/70246212>
	- 对象的大小按8字节对齐(String大小为24字节)
	- 浅堆大小和对象的内容无关，只和对象的结构有关
2. 深堆：
	- 一个对象被回收后，可以释放的真实内存大小
	- 只能通过(直接或者间接)对象访问到的所有对象的浅堆之和就是深堆(该对象支配树大小)
	- 释放该对象可能会将该对象支配的对象的内存也一起释放

**4)对象查询语言(OQL)：**
类似于SQL，具体可以参考博客：
<https://blog.csdn.net/Miklechun/article/details/42965139>

---
## 7.使用JVisualVM分析Dump堆
1. 打开：
【文件】-->【装入】选择备份文件导入即可：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-41.jpg)
2. 打开之后的主页面是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-42.jpg)
功能没有MAT丰富，很简洁。
3. 可以查看类信息或者和另一个备份文件比较：
![](http://p5ki4lhmo.bkt.clouddn.com/00062JVM%E5%AD%A6%E4%B9%A05-43.jpg)
4. 实例数这个测试代码没有产生，也很简单
OQL的使用可以查看前面介绍的博客
基本也就这些使用了

---
## 8.OOM分析的目的：
1. 找出OOM的原因
2. 推测系统OOM时的状态
3. 给出解决这个OOM的方法

---
