---
title: JVM学习笔记（六）：锁与字节码执行
date: 2018-5-17 23:05:11  
tags: JVM
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
	- Javap
	- 简单的字节码执行过程
	- 常用的字节码
	- 使用ASM生产Java字节码
	- JIT及相关参数

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