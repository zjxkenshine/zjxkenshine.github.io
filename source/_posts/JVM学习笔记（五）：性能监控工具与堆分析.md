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
	- OOM的原因
	- MAT使用基础
	- 使用Virtual VM分析堆
	- Tomcat OOM分析

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
`jmap -dump:live,format=b,file=D:/dump.txt 8580`
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
