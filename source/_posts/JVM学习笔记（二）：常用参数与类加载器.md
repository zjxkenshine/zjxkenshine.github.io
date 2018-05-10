---
title: JVM学习笔记（二）：常用参数配置及分配策略
date: 2018-5-7 22:37:00
tags: JVM
categories: Java基础

---
## 0.学习准备
1. 基础落下了很多了，是时候啃一波JVM了
感觉很深奥，就先从视频看起，以后再看书补充啦
2. 参考资料
参考书籍《深入理解Java虚拟机》，《Java虚拟机基础教程》
参考视频《深入理解JVM》(主要学习)
可以先学习GC再回来理解
3. JVM常用配置参数
	- 跟踪参数(Trace)
	- 堆的分配参数
	- 永久区分配参数(JDK1.8之后取消了永久代)
	- 栈的分配参数
	- 内存分配及回收策略

---
## 1.跟踪参数
**1)测试准备：**
1. 跟踪参数的使用：
项目目录如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-01-1.jpg)
创建项目之后会针对测试类进行配置。
Eclipse下RUN-->RUN configuration/DEBUG configuration
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-02.jpg)
2. TestGCArgs.java类代码如下：
		public class TestGCArgs {
			public static void main(String[] args) {
				String sum="";
				for(int i=0;i<5000;i++){
					String a="123";
					sum=sum+a;
				}
				System.out.println("finish");
			}
		}

### 对GC的跟踪
-verbose:gc：打开gc日志
1. 打印简要GC信息：`-XX:+PrintGC`
测试参数：-verbose:gc -XX:+PrintGC
测试结果：
		[GC (Allocation Failure)  45568K->704K(173056K), 0.0026189 secs]
		[GC (Allocation Failure)  46272K->686K(173056K), 0.0021835 secs]
		..省略数行..
		[GC (Allocation Failure)  182923K->760K(303616K), 0.0030116 secs]
		finish
结果分析：
		[GC (Allocation Failure)  45568K->704K(173056K), 0.0026189 secs]
		[GC （gc原因) GC前大小->GC后大小(堆大小)，所用时间]
Allocation Failure：
引起垃圾回收的原因. 本次GC是因为年轻代中没有任何合适的区域能够存放需要分配的数据结构而触发的。
2. 打印GC详细信息：`-XX:+PrintGCDetails`
测试参数：-verbose:gc -XX:+PrintGCDetails
测试结果：
		[GC (Allocation Failure) [PSYoungGen: 45568K->648K(52736K)] 45568K->656K(173056K), 0.0149955 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
		[GC (Allocation Failure) [PSYoungGen: 46216K->672K(98304K)] 46224K->680K(218624K), 0.0088765 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
		finish
PSYoungGen：代表时新生代代的GC
程序运行结束后还有一大段：堆的基本状况
		Heap
		//新生代信息
		 PSYoungGen      total 98304K, used 62667K [0x0000000785b80000, 0x000000078c280000, 0x00000007c0000000)
		//对象产生的空间
		  eden space 91136K, 68% used [0x0000000785b80000,0x000000078980ae70,0x000000078b480000)
		//from,to幸存代，大小一定相等
		  from space 7168K, 9% used [0x000000078bb80000,0x000000078bc28030,0x000000078c280000)
		  to   space 7168K, 0% used [0x000000078b480000,0x000000078b480000,0x000000078bb80000)
		//老年代的总信息
		 ParOldGen       total 120320K, used 8K [0x0000000711200000, 0x0000000718780000, 0x0000000785b80000)
		  object space 120320K, 0% used [0x0000000711200000,0x0000000711202000,0x0000000718780000)
		//元空间(jdk1.8版本用来替代永久代)
		 Metaspace       used 2665K, capacity 4486K, committed 4864K, reserved 1056768K
		  class space    used 288K, capacity 386K, committed 512K, reserved 1048576K
可能还会有永久代，共享区间等信息。
最后的那串数字：
		[0x0000000785b80000,0x000000078980ae70,0x000000078b480000)
		[低边界，当前边界，最大边界)
会发现分配的空间((最大边界-低边界)/1024/1024)比描述的空间要大。(学习GC算法时详细介绍)
3. 打印GC发生的时间戳：`-XX:+PrintGCTimeStamps`
测试参数：-verbose:gc -XX:+PrintGCTimeStamps
测试结果：
		0.570: [GC (Allocation Failure)  45568K->688K(173056K), 0.0038101 secs]
		0.620: [GC (Allocation Failure)  46256K->600K(173056K), 0.0106405 secs]
		0.673: [GC (Allocation Failure)  46168K->628K(173056K), 0.0017099 secs]
		finish
4. 重定向GC日志：`-Xloggc:位置`
以文件多的格式，可以帮助开发人员分析问题
5. 每次GC前后都输出堆信息(那一大段)：`-XX:+PrintHeapAtGC`
测试参数：-verbose:gc -XX:PrintHeapAtGC
每次GC前后都输出信息，而不是到最后再输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-03.jpg)

### 对类的监控参数
1. 创建类TraceClassArgs:
		public class TraceClassArgs {
			public static void main(String[] args) {
				Math.abs(-123);
				String a=new String();
				Random random=new Random();
				Date date=new Date();
				System.out.println("finish");
			}
		}
2. 监控类的加载：`-XX:+TraceClassLoading`
测试参数：-XX:+TraceClassLoading
每次运行都要导入N多个jdk的jar包：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-04.jpg)
3. `-XX:+PrintClassHistogram`打印类的信息(表)
开始不会打印任何类的信息
在程序运行时按下Ctrl+Break之后就会显示出类信息
测试参数：-XX:+PrintClassHistogram
我这里一致测试不出来，可能是参数错了，贴个视频的截图吧：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-05.jpg)

---
## 2.堆的分配参数
### 最大堆与最小堆
1. 测试代码：
		public class TestHeapArgs {
			public static void main(String[] args) {
				System.out.println("Xmx= "+Runtime.getRuntime().maxMemory()/1024/1024+"M");//最大可以使用空间
				System.out.println("free mem= "+Runtime.getRuntime().freeMemory()/1024/1024+"M");//可用的memory
				System.out.println("total mem= "+Runtime.getRuntime().totalMemory()/1024/1024+"M");//总memory，分配的内存
			}
		}
2. 指定最大堆和最小堆：`-Xmx -Xms`
最小堆就是打开虚拟机就分配的最小堆大小。
测试参数：-Xmx20m -Xms5m
测试结果：
		Xmx= 18M	//最大可用内存
		free mem= 4M	//可用内存
		total mem= 5M	//总内存，目前只分配了最小的
3. 在上面代码的最前面添加代码，分配1M空间：
		byte[] b=new byte[1*1024*1024];
测试结果：
		Xmx= 18M
		free mem= 3M
		total mem= 5M
可用内存变为了3M
4. 将分配内存增大：
		byte[] b=new byte[5*1024*1024];
测试结果：
		Xmx= 18M
		free mem= 5M
		total mem= 10M	//又分配了5M的内存
在前面代码(分配5M的前面加上GC)
		System.gc();
会发现空闲空间变大了一点。
5. 堆分配的一些思考：
`-Xmx -Xms`应该如何设置
如何将JRE瘦身，只保留自己用到的类

### 新生代相关的配置参数
**1)参数简介：**
1. `-Xmn`：设置新生代大小
2. `-XX:NewRatio`：
	- 新生代(eden+2*s)和老年代(不包含永久区)的比值
	- 例：4，表示新生代：老年代=1：4，即年轻代占堆的1/5
3. `-XX:SurvivorRatio`：
	- 设置两个存活区和eden的比值(两个存活区总大小+eden=新生代大小)
	- 例：8，表示一个Survivor:eden=1:8,所以一个存存活区占新生代大小的1/(8+1+1)=1/10.

**2)-Xmn参数测试：**
1. 测试一：
测试代码：
		public class HeapTest01 {
			public static void main(String[] args) {
				byte[] b =null;
				for (int i = 0; i < 10; i++) {
					b=new byte[1*1024*1024];	//分配10M空间
				}
			}
		}
测试参数：(新生代大小设置为1m)
		-Xmx20m -Xms20m -Xmn1m -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-06.jpg)
此时因为新生代非常小，所有的数据都在老年代分配。(老年代使用率54%)
2. 测试二：
测试代码同上。
测试参数：(新生代大小设置为15m)
		-Xmx20m -Xms20m -Xmn15m -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-07.jpg)
所有的数据都在新生代分配（老年代使用率0%,没有触发GC）
3. 测试三：
测试代码不变
测试参数：(新生代大小设置为7m)
		-Xmx20m -Xms20m -Xmn7m -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-08.jpg)
触发了一次新生代GC,回收时发现存活区(幸存代)太小，无法进行回收，需要老年代的保护。

**3)其他两个参数测试：**
1. 测试四：
测试代码同上面的不变
测试参数：(幸存代设置为1/4)
		-Xmx20m -Xms20m -Xmn7m -XX:SurvivorRatio=2 -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-09.jpg)
触发了三次新生代的GC,幸存代够用，所以未使用老年代的对象空间。(老年代对象的120k的使用是系统级别的对象空间)
2. 测试五：
测试代码不变。
测试参数：(新生代：老年代=1:1)
		-Xmx20m -Xms20m -XX:NewRatio=1 -XX:SurvivorRatio=2 -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-10.jpg)
所有的GC都在新生代，对象没有使用老年代空间
但是幸存代的使用率只有63%,会浪费空间。
3. 测试六：
测试代码不变。
测试参数：(调整幸存代大小)
		-Xmx20m -Xms20m -XX:NewRatio=1 -XX:SurvivorRatio=3 -XX:+PrintGCDetails
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-11.jpg)
GC次数从两次减少为一次，且幸存区使用率高，未占用老年代空间。

### OOM时堆的常用参数
**1)导出OOM到文件：**
1. OOM时导出堆到文件：`-XX:+HeapDumpOutOfMemoryError`
2. 导出OOM的路径：`-XX:+HeapDumpPath`
`-XX:+HeapDumpPath=d:/a.dump`
3. 也可以重现异常，但是导出堆文件更加好。
示例例参数：
		-Xmx20m -Xms5m -XX:NewRatio=1 -XX:+HeapDumpOutOfMemoryError -XX:+HeapDumpPath=d:/dump
dump出来的文件大小一般和最大堆的大小是保持一致的。
4. 视频中的例子：
代码及运行结果：(参数就是上面的那个)
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-12.jpg)
dump文件中的内容：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-13.jpg)

**2)一个OOM时很有用的参数：**
1. `-XX:OnOutOfMemoryError`
2. 可以在OOM时执行一个脚本
	如：-XX:OnOutOfMemoryError=D:/tools/jdk7/printstack.bat %p
	%p代表当前java程序的pid
	printstack.bat 会将信息导出到一个txt文件(D:/a.txt)
	程序OOM时，在D:/a.txt中会生成线程的dump
3. 脚本可以任意指定，所以还可以用来发送警告，发送邮件甚至是重启程序。

---
## 3.堆分配参数的总结
- 根据实际情况调整新生代和幸存代的大小
- 官方推荐新生代占堆的3/8,幸存区占新生代的1/10
- 在OOM时记得DUMP出堆，确保可以排查现场问题

---
## 4.永久区分配参数
1. 设置永久区的初始空间大小：`+XX:PermSize`
设置永久区的最大空间大小：`+XX:MaxPermSize`
类似于最大堆与最小堆的关系。
表示一个系统中最多可以容纳多少个类型。
2. 使用CGlib等动态代理时会产生大量的类，这些类可能撑爆永久区产生OOM。
视频中的测试代码：参数默认
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-14.jpg)
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-15.jpg)
3. 上述例子中，堆空间没有使用完，永久区溢出也一样会报OOM。

---
## 5.栈大小分配
1. `-Xss`:
	- 通常只有几百kb
	- 决定了函数调用的深度
	- 每个线程都有独立的栈空间
	- 局部变量(表)，参数分配在栈上
2. 很少会把栈的大小调到很大的程度。
3. 如果想在系统中多跑一些线程，那么可以将栈空间减小，
但是栈空间又决定了函数调用的深度
4. 视频中的测试：一个递归(导致栈溢出)
![](http://p5ki4lhmo.bkt.clouddn.com/00058JVM%E5%AD%A6%E4%B9%A02-16.jpg)
调用深度从701次变为了1817次。
5. 能增加调用深度的另一个方法就是减少参数(局部变量)。

---