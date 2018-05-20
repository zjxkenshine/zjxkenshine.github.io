---
title: JVM学习笔记（六）：锁与字节码执行
date: 2018-5-17 23:05:11  
tags: 
- JVM
- 并发
categories: Java基础

---
## 0.学习准备
1. 参考资料
参考书籍《深入理解Java虚拟机》
参考视频《深入理解JVM》(目前学习)
2. 简单目录：
	- 线程安全
	- 对象头Mark
	- JVM层面的锁优化：
		- 偏向锁
		- 轻量级锁
		- 自旋锁
	- Java层面的锁优化：
		- 减少锁持有时间
		- 减小锁粒度
		- 锁分离
		- 锁粗化
		- 锁消除
		- 无锁
	- javap
	- 简单的字节码执行过程
	- 常用的字节码
	- 使用ASM生成Java字节码
	- JIT即时编译和相关参数

---
## 1.线程安全
1. 相关的详细内容和例子可以查看多线程相关的笔记：
<https://zjxkenshine.github.io/tags/%E5%A4%9A%E7%BA%BF%E7%A8%8B/>
2. 测试代码：
模拟一个累加的功能(如投票等)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-01.jpg)
测试类及输出结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-02.jpg)
出错原因：
ArraList进行扩展时另一个线程尝试进行插入，此时的ArrayList不可用，所以会抛出越界异常。

---
## 2.对象头Mark
1. Mark Word,对象头的标记，32位
2. 描述对象的hash，锁信息，垃圾回收标记，年龄等
	- 指向锁记录的指针
	- 指向monitor的指针
	- GC标记
	- 偏向锁线程ID

---
## 3.偏向锁，轻量级锁和自旋锁
**1)偏向锁：**
1. 偏向锁简介及使用场景：
	- 只有在竞争不激烈的场合才能够使用偏向锁来提高性能
	- 在竞争激烈的场合偏向锁会增加系统负担
	- 锁会偏向当前已经占有锁的线程
	- 将对象头Mark的标记设置为偏向，并将线程ID写入对象头
	- 只要没有进程，获得偏向锁的线程在将来进入同步块时不需要同步
	- 其他线程请求相同的锁时，偏向模式结束
2. 偏向锁的使用参数：
	- `-XX:+UseBiasedLocking`
	- 默认启用
3. 简单例子：
测试代码：(Vector自带同步方法)
		public class BasedLockTest {
			public static List<Integer> numberlist=new Vector<Integer>(); 
			public static void main(String[] args) {
				long begin =System.currentTimeMillis();
				int count=0;
				int startnum=0;
				while(count<10000000){
					numberlist.add(startnum);
					startnum+=2;
					count++;
				}
				long end =System.currentTimeMillis();
				System.out.println(end-begin+"ms");
			}
		}
测试参数1：默认参数，使用偏向锁
测试结果1：
		7164ms
测试参数2：使偏向锁在程序启动时就启用
		-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0
测试结果2：
		6995ms
测试参数3：关闭自旋锁
		-XX:-UseBiasedLocking
测试结果3：
		7271ms
4. 测试结论：
在这种单线程无竞争的情况下使用偏向锁能够提高系统性能。

**2)轻量级锁：**
1. BasicObjectLock：存放在线程栈中的锁对象
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-03.jpg)
包含了两部分：
	- BasicLock:存放了对象头信息
	- 指向持有了该锁对象的指针
2. 普通的锁(重量级锁)处理性能不够理想，
轻量级锁是一种快速的锁定方法
3. 轻量级锁的加锁过程：
如果对象没有上锁：进行两步操作(交换)
	- 将对象头的Mark指针保存到锁对象BasicObjectLock中
	- 将对象头设置为指向锁的指针(指向栈空间)
4. 简单的代码示例如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-04.jpg)
5. 轻量级锁的使用环境：
	- 如果轻量级锁失败，说明存在竞争，升级为重量级锁(常规锁)
	- 没有锁竞争的时候，可以减少传统锁使用OS互斥量产生的性能损耗
	- 竞争激烈的时候轻量级锁会做很多额外的操作，影响性能
6. 轻量级锁是默认开启的，可以使用如下参数同时关闭轻量级锁和偏向锁：
		-XX:+UseHeavyMonitors

**3)自旋锁：**
1. 简介：
当竞争存在时，如果线程可以很快获得锁，那么可以不再OS层挂起线程，可以让线程做几个空操作(自旋)
2. 使用参数：
JDK1.6：`-XX:UseSpining`
JDK1.7去掉了这一参数，改为内置实现，默认开启。
3. 注意事项：
	- 同步块很长，自旋失败，会影响系统性能
	- 同步块很短，自旋成功，节省系统挂起切换的过程，提升系统性能
	- 只要空转指令的开销小于挂起和切换的开销自旋就是成功的
	自旋之后还拿不到锁，还是需要挂起和切换线程，那么就是自旋失败的

**4)三种锁的总结：**
1. 并不是Java语言层面的锁优化方法，是JVM层面的
2. 内置于JVM中的锁的优化方法和获取锁的步骤：
	- 偏向锁可用会先尝试偏向锁
	- 轻量级锁可用会先尝试轻量级锁
	- 以上两种锁都失败则尝试自旋锁
	- 自旋锁失败则尝试普通锁，使用OS的互斥量在操作系统层挂起

---
## 4.Java语言层面进行锁优化
简单目录：
- 减少锁持有时间
- 减小锁粒度
- 锁分离
- 锁粗化
- 锁消除
- 无锁

**1)减少锁的持有时间：**
1. 没有必要做同步的代码就不要放在同步代码块或者同步方法中。
2. 优化前的代码：
		public synchronized void syncMethod(){
			System.out.println("无关代码块1");
			System.out.println("需要同步的代码");
			System.out.println("无关代码块2");
		}
3. 优化后的代码：
		public void syncMethod(){
			System.out.println("无关代码块1");
			synchronized(this){
				System.out.println("需要同步的代码");	
			}
			System.out.println("无关代码块2");
		}
4. 锁持有时间减少，偏向锁的成功率提高。

**2)减小锁粒度：**
1. 基本思想：
将大对象拆成小对象，大大提升并行度，降低锁竞争
锁竞争降低，偏向锁和轻量级锁的成功率提高
2. 典型实现：ConcrrentHashMap（稍后再介绍）
3. HashMap的同步实现：
	- Collections.synchronizedMap(Map<k,v> m)
	获得一个同步hash表
	- 返回一个synchronizedMap对象，内部的get,put实现如下：
	![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-05.jpg)
4. ConcrrentHashMap减小锁粒度的实现：
	- 分为若干个段(Segment)：`Segment<K,V>[] segments`
	- Segment中维护HashEntry<K,V>（表项）
	- put操作时：
	先定位到相应的Segment，再锁定这个Segment，再执行put操作
	- 减小锁粒度之后，ConcrrentHashMap允许多个线程同时进入。
5. 如果锁粒度太细也会引起性能的损耗

**3)锁分离：读锁和写锁**
1. 广义上说也是属于减小锁粒度的一种
减小锁粒度：结构上对对象做分离
锁分离：功能上对锁做分离
2. ReadWriteLock：读写锁
在读多写少的情况可以提高系统的性能。
读读不互斥，读写互斥，写写互斥：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-06.jpg)
3. 读写分离的思想扩散：
只要操作互相不影响，锁就可以分离。
4. LinkedBlockingQueue：阻塞链表(队列)
结构示意图如下:
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-07.jpg)
take操作和put操作是互不影响的，可以将这两种操作的锁分离。

**4)锁粗化：**
1. 通常情况下，为了保证多线程的有效并发，会要求每个线程是有锁的时间尽可能少，即使用完公共资源之后立即释放锁。等待这个锁的其他线程就能尽早得到公共资源。
2. 但是物极必反，如果对一个锁一直进行请求，同步和释放，也会消耗系统性能，影响锁的优化。在这种场合就需要泛起到而行之。
3. 示例情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-08.jpg)
粗化后的代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-09.jpg)
4. 极端情况：循环中获取锁
		for(int i=0;i<n;i++){
			synchronized(lock){
				//同步代码
			}
		}
一般情况下是不合道理的，粗化过后的代码：
		synchronized(lock){
			for(int i=0;i<n;i++){
				//同步代码
			}
		}

**5)锁消除：**
1. 是一种JVM层面的优化
2. 在即时编译时如果发现不可能被共享的锁对象，就消除这个对象的锁。
一般情况下程序员是不会给一些完全不可能同步的对象加锁的。
但是StringBuffer和Verctor等的类内部有锁，在使用这些类对象的时候就隐式的将锁引入到其中了。
3. 如以下代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-10.jpg)
StringBuffer的append操作自带了锁，但是在这段代码中完全没有用到锁。
关闭和不关闭锁消除的运行结果测试：循环2000000次
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-11.jpg)

**6)无锁：**
1. 无锁是最好的一种锁方式
2. 锁是一种悲观操作：预期竞争是一定存在的
无锁是一种乐观的操作：预期竞争是不存在的
3. 锁是在操作之前先定义好如何解决竞争。
无锁是在操作之后遇到同步问题再回来定义解决。
4. 无锁的一种实现：CAS
	- CAS(Compare And Swap)，比较交换技术
	- 非阻塞的同步
	- CAS(V,E,N)：
		- V：要更新变量
		- E：期望值
		- N：新值
		- 如果新值满足期望(V=E)，那么久赋值给变量V，做完以后将V值返回。
	- 在应用层面判断是否被多线程干扰，如果有干扰，则通知线程重试
	不停重试直到成功，或者放弃
	- 比较交换是一个cpu指令。
5. 无锁操作会使程序变复杂，但是会使性能变的更好。
6. JUC并发包中的原子操作就是一种CAS无锁的实现：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-12.jpg)
7. java.util.concurrent.atomic包中的都是无锁实现，性能高于一般的有锁操作。

---
## 5.javap
class文件[反汇编](https://baike.baidu.com/item/%E5%8F%8D%E6%B1%87%E7%BC%96/10858476?fr=aladdin)工具

**1)javap的简单使用示例：**
1. Calc类代码如下：
		public class Calc{
			public int calc(){
				int a=500;
				int b=200;
				int c=50;
				return (a+b)/c;
			}
		}
2. 使用命令：`javap -verbose Calc`
得到对应的反汇编的方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-13.jpg)
3. 汇编方法解释：
前面是一些堆栈参数信息。
第一列的数字为字节码的偏移量（行号）
第二列为指令，第三列为指令的参数
4. 关于指令(字节码)：
	- sipush：将操作数压栈(一个短整型常量)
	- istore_n：弹出一个栈帧，放入局部变量表的第n个位置
	- binpush：将一个单字节的常量(-128~127)推送至操作数栈顶
	- iload_n：将局部变量表的第n个位置的数压入操作数栈
	- iadd：相加
	- idiv：相除
5. 每一个指令都是有相对应的字节码的

**2)简单的字节码执行过程：**
1. 压入操作数500：(0,3)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-14.jpg)
2. 压入操作数200：(4,7)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-15.jpg)
3. 压入单字节操作数50：(8,10)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-16.jpg)
4. 取出局部变量1，2：(11,12)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-17.jpg)
5. 相加并添加局部变量3：(13,14)
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-18.jpg)
6. 相除并返回：（15，16）
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-19.jpg)

---
## 6.关于字节码
**1)字节码简介：**
1. 字节码在Class文件中的位置：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-28.jpg)
该类文件中的`2A 1B B5 00 20 B1`就是字节码。
2. 字节码指令为一个byte类型的整数：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-20.jpg)
左边的是便于人们理解的指令名称，右边的则是在计算机当中的表示方式。
3. 关于上图`2A 1B B5 00 20 B1`字节码的理解：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-21.jpg)

**2)Java中常用的字节码：**
1. 常量入栈的字节码：
JVM没有寄存器，所有的操作都要通过栈来完成
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-22.jpg)
2. 局部变量压栈：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-23.jpg)
3. 出栈装入局部变量：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-24.jpg)

**3)通用型的栈操作指令：**
1. 通用栈操作(无类型)：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-25.jpg)
2. 类型转换字节码：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-26.jpg)
以i2l为例：
	- 将int类型转换为long类型
	- 执行前，栈：`...,value`
	- 执行后，栈：`...,result.word1,result.word2`
	(long需要两个字空间)
	- 弹出int，扩展为long，并入栈。
3. 运算操作：
分为整数的运算和浮点型的运算
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-27.jpg)
最前面的ilfd代表的是数据类型。
都是一些基本的加减乘除的操作。

**4)对象，流程控制相关的字节码：**
1. 对象操作指令：
	- new：新建对象
	- getfield：得到给定实例对象的值
	- putfield：设置给定实例对象的值
	- getstatic：获得静态对象
	- putstatic：设置静态对象
2. 流程控制字节码：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-28.jpg)
如ifeq byte1 byte2：
	- 执行前，栈：...,value
	- 执行前，栈：...
	- value出栈后如果栈顶元素为0则跳转到byte1，byte2指定的字节码处
	`(byte1<<8)|byte2`,byte左移8为按位或byte2
3. 方法调用和返回：
	- invokevirtual：对普通的实例方法进调用(动态绑定，有多态)
	- invokespecial：对子类调用父类方法等场合进行调用(静态绑定，无多态)
	- invokestatic：对静态方法进行调用
	- invokeinterface：调用接口方法的指令
	- xreturn：统一返回指令(x可以为ilfda或者空)

---
## 7.使用ASM生成Java字节码
1. asm是java的字节码操作框架，可以动态查看类的信息，动态修改，删除，增加类的方法。
可以用于修改现有的类或者动态产生新的类。
2. 目前许多框架如cglib、Hibernate、Spring都直接或间接地使用ASM操作字节码
还有一些软件如Eclipse也使用了ASM操作字节码
3. 关于ASM的具体如何使用以后再补充，
可以参考博客:
[asm字节码操作 方法的动态修改增加](https://blog.csdn.net/saifeng/article/details/46238387)
4. ASM可以使先类似于AOP的织入功能
	- 在函数开始或者结束时嵌入一些字节码
	- 可用于鉴权，日志等

---
##8.JIT相关参数：
**1)JIT简介：**
1. 在Java当中，单纯的字节码执行的效率是很差的(解释执行)，所以需要对热点代码进行编译，编译成机器码再执行。在运行时编译叫做JIT(Just-In-Time)
2. JIT的基本思路是将热点代码（执行比较频繁的代码），编译成机器码。
以解释为基础，对热点进行编译。
3. 编译技术大约分为两种，一种AOT，只线下（offline）就将源代码编译成目标机器码,这是普遍用在系统程序语言中；另一种是JIT，只及时的编译，但是大部分的JIT引擎，针对的是将IR（中间代码,如JavaByteCode) 在运行时, 有针对性的翻译成机器码。
4. JIT示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00064JVM%E5%AD%A6%E4%B9%A06-29.jpg)
是否是热点由方法调用计数器和回边计数器来控制：
	- 调用计数器：方法调用次数
	- 回边计数器：方法内循环次数

**2)JIT相关参数：**
1. 可以使用如下JVM参数对编译的方法进行打印：
		-XX:CompaileThreshold=1000 -XX:+PrintCompilation
2. 三个参数：
-Xint：全部解释执行
-Xcomp：全部编译执行
-Xmixed：混合执行，默认的
3. 关于更多的编译的知识，参考《编译原理》。

---