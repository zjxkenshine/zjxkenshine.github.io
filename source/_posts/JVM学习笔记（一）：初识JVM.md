---
title: JVM学习笔记（一）：JVM简介及JVM运行机制
date: 2018-4-28 22:56:08  
tags: 
- JVM
- Java
categories: Java基础

---
## 0.学习环境
1. 基础落下了很多了，是时候啃一波JVM了
感觉很深奥，就先从视频看起，以后再看书补充啦
2. 参考资料
参考书籍《深入理解Java虚拟机》
参考视频《深入理解JVM》(主要学习)
3. JVM原理
	- JVM的概念
	- JVM的发展史
	- JVM的种类
	- Java语言规范
	- JVM规范
4. JVM运行机制
	- JVM启动流程
	- JVM基本结构
	- 内存模型
	- 编译和解释运行的概念

---
## 1.JVM的概念
**1)虚拟机简介：**
1. JVM：
Java Virtual Machine的简称。意为Java虚拟机。
2. 虚拟机：
指通过软件模拟的具有完整硬件功能的、运行在一个完全隔离的环境中的完整计算机系统。
3. 常见的虚拟机：
	- JVM
	- VMwave
	- Virtual Box

**2)JVM和其他两个虚拟机的区别：**
1. VMwave与VirtualBox是通过软件模拟物理CPU的指令集
物理系统中会有很多的寄存器
2. JVM则是通过软件模拟Java字节码的指令集
JVM中只是主要保留了PC寄存器，其他的寄存器都进行了裁剪
3. JVM是一台被定制过的现实当中不存在的计算机。

---
## 2.JVM和Java的发展史
**1)20世纪Java发展：**
1. 1996年SUN JDK 1.0时发布：Classic VM
	- 纯解释运行，使用外挂进行JIT(编译器)
2. 1997年JDK 1.1发布：
	- AWT、内部类、JDBC、RMI、反射(Java的核心)
	- RMI:远程方法调用(Remote Method Invocation)。能够让在某个java虚拟机上的对象像调用本地对象一样调用另一个java 虚拟机中的对象上的方法。
3. 1998年JDK 1.2发布：Solary Exact VM(仅存在了很短的时间)
	- JIT和解释器混合执行
	- Accurate Memory Management 精确内存管理，数据类型敏感
	- 提升GC性能
4. JDK1.2开始成为Java2(J2SE,J2EE,J2ME出现)
并且加入了Swing Collection。

**2)二零零几年：**
1. 2000年JDK1.3：Hotsport作为默认虚拟机发布 
JDK1.3加入了JavaSound
2. 2002年JDK1.4：Classic VM退出历史舞台
1.4更新内容：Assert，正则表达式，NIO，IPV6，日志API，加密类库，异常链，XML解析器等。
3. 2004年JDK1.5即JDK5,Java5（很重要的一个版本）
Java5更新内容：泛型，注解，装箱，枚举，可变长参数，Foreach循环。
虚拟机层面：改进了Java内存模型(JMM)，提供了JUC并发包。
4. JDK1.6 java6：
更新内容：脚本编程的支持(动态语言支持)，jdbc4.0，Java编译器API，微型Http服务器API等。
虚拟机层面：锁与同步，垃圾收集，类加载等算法的改动
5. JDK1.6发布后由于一系列原因，JDK有好几年都没有大更新。JDK1.6一共有37个Update版本。

**3)二零一几年：**
1. 2011年JDK1.7/Java7发布：
指定了十个里程碑但是始终无法完成，未完成的项目推到Java8(模块化则推到了Java9)。
	- G1收集器(Update4才正式发布)
	- 加强对非Java语言的调用支持
	- 升级类加载器架构
	- 64位系统压缩指针
	- NIO2.0
2. 2014年Java8发布：
	- Lamda表达式
	- 语法增强
	- Java类型注释等
3. 2016年Java9发布：
	- 模块化

**4)Java与JVM发展历史中的大事件：**
1. 使用最广泛的JVM为HotSpot
2. HotSpot最早为Longview Technologies开发，被SUN收购
3. 2006年，Java开源，并建立OpenJDK
	- HotSpot成为SUN JDK和OpenJDK中所带的虚拟机
4. 2008年，Oracle收购BEA
	- 得到JRockit VM
5. 2010年Oracle收购SUN
	- 得到HotSpot
6. Orcale宣布在JDK8时整合HotSpot和JRockit VM，优势互补
	- 在HotSpot的基础上移植JRockit的优秀特性

---
## 3.JVM的种类
1. KVM:
	- SUN发布
	- IOS和Android兴起之前，广泛应用与手机系统
2. CDC/CLDC HotSpot
	- 手机、电子书、PDA等设备上的统一Java编程接口
	- J2ME的重要组成部分
	- 同样在IOS和Android兴起之后受到了很大的挑战
3. JRockit:
	- BEA
4. IBM J9 VM：
	- 仅在IBM内部使用
5. Apache Harmony
	- 兼容JDK1.5和1.6的Java程序运行平台，开源
	- 与Orcale关系恶劣，退出JCP，Java社区分裂
	- OpenJDK出现后受到挑战，于2011年退役
	- 没有大规模的商用经历
	- 对Android的发展又积极作用

---
## 4.Java及JVM规范
Java语言规范：
- 语法
- 变量
- 类型
- 文法

JVM规范：
- Class文件类型
- 运行时数据
- 帧栈
- 虚拟机的启动
- 虚拟机的指令集

**1)Java语言规范：**
1. 语法：
![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-01.jpg)
2. 词法结构：
	- \u+4个16进制数字，表示UTF-16
	- 行终结符：CR,or LF,or CR LF
	- 空白符的定义：
	空格 tab \t(换页) \f(行终结符)
	- 注释
	- 标识符：就是变量名啊那种
	不能是布尔型，关键字，Null
	- 关键字 
3. 词法结构中的数值的表示：允许使用下划线
	- int：
	如：0 2 0237 0xDada_cafe 0x00_FF__00_FF
	- long：
	如： 0l 2L 0x1000000l 2_17_483_648L 0xC0B0L
	- float：
	如：1e1f 2.f .3f 3.14f 6.66666e+23f
	- double
	如：1e1f 2. .3 3.14 1e-9d 1e137
	- 操作：
	如：+= -= *= /= &= |= ^= %= < <= > >= ...
4. 类型和变量：
	- 元类型：
	byte，short，int，long，float，char
	- 变量初始值：
		- boolean false
		- char \u0000
	- 泛型：
5. 元数据类型和对象(封装类型)的差异：
![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-02.jpg)
6. 其他语言规范：
	- Java内存模型
	- 类加载链接的过程
	- public final static abstract的定义
	- 异常
	- 数组的使用
	- 。。。
7. Java规范定义了什么是Java语言

### JVM规范
**2)JVM规范：**
1. Java语言和JVM是相对独立的：
运行在JVM上的语言还有：Groovy，Clojure，Scala
能在JVM上运行说明这些语言都是符合JVM规范的。
2. JVM只要定义了二进制class文件和JVM指令集等。
3. JVM规范主要定义的内容：
	- Class文件格式
	- 数字的内部表示和存储：(Java中没有无符号数)
	如：Byte -128 to 127(-2^7到2^7-1)
	- ReturnAddress数据类型定义：
	指向操作码的指针。不对应Java的数据类型，不能在运行时修改。Finally实现需要。(在Java中找不到该数据类型)
	- 定义PC
	- 堆
	- 栈
	- 方法区
4. JVM当中整数的表达：
	- 原码：第一位符号位（0为正数，1为负数）
	- 反码：符号位不动，原码取反
	- 负数补码：符号位不动，反码加一
	- 正数补码：与原码相同
5. 打印出整数在二进制当中的表示：
		int a=-6;想要打印为二进制的整数
		for(int i=0;i<32;i++){ 	//整数32位，每次取一位
			int t=(a&0x800000000>>>i)>>>(31-i);
			System.out.print(t);
		}
0x800000000代表最高位为1的数字。
6. 为什么要使用补码：
	- 0的表示：不管算正算负补码都是一样的(无歧义表示0)
	![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-03.jpg)
	- 补码能很好的参与计算机之间的计算：
	![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-04.jpg)
7. Float的表示与定义：支持IEEE754
	- `s eeeeeeee (x)mmmmmmmmmmmmmmmmmmmmmmm`
	s:符号位；e:8位指数位；m:23尾数位;x:附加位(不写出，由JVM给出)
	- 尾数附加位的值由指数位确定：
		- 指数位e全为0，则尾数附加位为0，否则尾数附加位为1
	- 最后的数值：e要进行位移
	`s*m*2^(e-127)`
	- 如-5的表示：
	![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-05.jpg)
8. 一些特殊的方法：
	- `<init>`:默认的构造函数(无参)
	- `<clinit>`:类的初始化方法
9. JVM指令集：
	- 类型转换：l2i(长整型转整型)
	- 出栈入栈操作：(JVM中没有寄存器，数据传递依靠栈，所以又很多栈的指令)
	`aload`,`astore`
	- 运算：`iadd`,`isub`等
	- 流程控制：`ifeq`相等，`ifne`不相等
	- 函数调用：
	`invokevirtual`、`invokeinterface`、`invokespecial`、`invokestatic`等
10. JVM对Java Library提供以下支持(Java无法实现)：
	- 反射 java.lang.reflect
	- 类加载器 ClassLoader
	- 初始化class和interface
	- 安全相关 java.security
	- 多线程
	- 弱引用
11. JVM的编译与反编译：
	- 源码到JVM指令的对应格式
	- javap
	- JVM反汇编格式：
	```
	<index> <opcode> [<operand1>[<operand2>...]] [comment]
	操作码    操作 		操作数   				    注解
	```
12. 反编译的例子：一个for循环的执行过程
![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-06.jpg)

---
## 5.JVM的启动流程
简单流程图：
![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-07-1.jpg)
1. 第一步：启动JVM——Java或者Javaw命令后面跟一个启动类
JAVA XXX,启动类中一般都有一个main方法。
2. 第二步：装载配置
会根据当前路径和系统版本寻找jvm.conf文件，找到文件之后会去定位所需要的dll文件。
3. 第三步：根据配置寻找JVM.dll
需要寻找的是匹配当前系统版本的dll
JVM.dll为JVM的主要实现
使用JVM.dll初始化虚拟机，获得相关的一系列native调用接口。
4. 第四步：初始化JVM，获得JNIEnv接口。
JNIEnv为JVM接口，findClass等操作通过它实现。
5. 第五步：找到Main方法并允许

---
## 6.JVM基本结构简介
简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00053JVM%E5%AD%A6%E4%B9%A01-08.jpg)
1. 类加载器子系统：ClassLoader
将Class文件加载到JVMz中
2. 本地方法栈的内容暂时不介绍
3. PC寄存器：又称为程序计数器
虚拟机运行时需要有指针指向下一条指令的地址。
4. 执行引擎：执行类的字节码
5. 垃圾收集器：很重要的一个模块

### pc寄存器，方法区与Java堆
**1)pc寄存器** 
- 又称为程序计数器
- 每个线程都有一个PC寄存器
- 在线程创建时创建
- 总是指向下一条指令的地址
- 执行本地方法时，PC寄存器的值为空(undefined)

**2)方法区：**
1. 保存装载的类的信息：
	- 常量(除字符串常量)
	- 静态变量
	- 字段方法信息
	- 编译后的字节码
2. 有一个别名叫Non-Heap(非堆)
3. JDK6时String常量信息置于方法，JDK7时已经移动到了堆。
4. 方法区经常被称为"永久区"，仅仅是因为涉及团队将GC分代收集扩展至方法区(使用永久代实现的方法区)，保存一些相对稳定的信息。
5. 所有线程共享的内存区域，由虚拟机维护。

**3)Java堆：**
1. 和程序开发最为密切的一个区间
2. 简介：
	- 和程序开发密切相关
	- 应用系统对象都保存在Java堆中
	- 所有线程共享的Java堆
	- 对分代GC来说，堆也是分代的
	- GC的主要工作区间

---
## 7.Java栈


---