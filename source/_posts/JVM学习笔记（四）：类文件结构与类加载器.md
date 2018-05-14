---
title: JVM学习笔记（四）：类文件结构与类加载器
date: 2018-5-13 15:08:59 
tags: 
- JVM
- 类
categories: Java基础

---
## 0.学习准备
1. 参考资料
参考书籍《深入理解Java虚拟机》
第三章：垃圾收集器与内存分配策略
参考视频《深入理解JVM》(目前学习)
2. 简单目录：
	- 语言无关性
	- 类文件的结构：(类如何组织和使用)
		- 魔数，版本号，访问符
		- 常量池
		- 超类，接口
		- 字段，方法，属性
	- 类装载验证流程
	- 什么是类装载器ClassLoader
	- JDK中ClassLoader默认设计模式
	- 热替换

---
## 1.JVM语言无关性
1. JVM运行的是Java编译而来的,但是并非只有Java的类文件能编译为.class文件。
很多种语言都能在JVM上运行。
2. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-01.jpg)
3. Class文件是JVM的一个基石。
如果要对JVM做深入理解，一定要理解Class文件。

---
## 2.类文件结构
1. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-02.jpg)
2. 介绍：
	- 类型：主要包括两种数据类型
		- 无符号数u：基本数据类型，u后面的数字代表多少位
		- 表：以"_info"结尾，由多个无符号数或者其他表作为数据项构成符合数据类型
	- 名称：从上往下
		- magic：魔数，是否是一个Java文件
		- minor_version：小版本
		- major_version：大版本
		- constant_pool_count：常量池内容个数
		- constant_pool：常量池具体内容
		- access flags：访问修饰符
		- this_class：当前类
		- super_class：超类，父类
		- interfaces_count：接口数量
		- interfaces：接口的信息
		- fields_count：字段数量
		- fields：字段信息
		- methods_count：方法数量
		- methods：方法信息
		- attributes_count：属性数量
		- attributes：属性信息
	- 数量：对应的数量
		- constant_pool_count表示常量的个数，constant_pool是存放的是所有常量信息的表项，存放的个数为constant_pool_count - 1，constant_pool可以将理解为一个数组，其中的每一项代表一个常量。
		- 因为Java只能单继承，所以超类数量为1。
3. 对应ClassFile的代码如下：
		ClassFile {
			u4 magic; 
			u2 minor_version; 
			u2 major_version; 
			u2 constant_pool_count; 
			cp_info constant_pool[constant_pool_count-1];
			u2 access_flags; 
			u2 this_class; 
			u2 super_class; 
			u2 interfaces_count; 
			u2 interfaces[interfaces_count]; 
			u2 fields_count; 
			field_info fields[fields_count]; 
			u2 methods_count; 
			method_info methods[methods_count]; 
			u2 attributes_count; 
			attribute_info attributes[attributes_count]; 
		}

### 魔数，版本号，标示符，
**1)魔数：**
1. magic：
又叫magic number，魔数
2. 值为四字节无符号数：
0xCAFEBABE(“咖啡宝贝”)
如果不是这个值，则说明不是一个类文件。

**2)版本号：**
1. 分为主版本号u2和次版本号u2(大版本号和小版本号)
记录的是用哪个版本的Javac编译的文件
2. 各版本编译器的版本号示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-03.jpg)
3. `-target`参数则可以使用目标版本的jdk来进行编译。

**3)访问修饰符(标示符)：**
1. 通常占两个字节u2
2. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-12.jpg)

### 常量池，类，接口
**1)常量池简介：**
1. `constant_pool_count`：常量池常量个数，u2
`constant_pool`：常量池的常量信息，可以认为是一个数组，最大下标constant_pool_count-1
2. 常量池信息表cp_info的部分类型：(类型,标志与描述）
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-04.jpg)
常量池中的每一种数据类型都有自己的数据结构。
3. 常量池中主要存放的是字面量和符号引用

**2)一些重要的常量类型：**
1. CONSTANT_Utf8：其实是一个utf8编码的String(由三部分组成)
----tag 1：标志
----length u2：byte数组长度
----bytes[length]：byte数组
结构示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-05.jpg)
`()I`：表示返回值为整型的方法
`()Ljava/lang/Object`：表示返回值为Objecet的无参方法
2.  CONSTANT_Integer：整型的字面量
----tag 3：标志
----byte u4：四个字节的值
结构示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-06.jpg)
3. CONSTANT_String：String类型字面值的引用
----tag 8
----string_index u2：指向utf8的索引(实际的值存储在utf8的字符串中)
结构示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-07.jpg)
4. CONSTANT_NameAndType：名字和类型引用
----tag 12
----name_index u2：名字的索引，指向utf8
----descripter_index u2：描述符类型索引，指向utf8
结构示例：默认构造函数的表示
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-08.jpg)
`()V`：表示无参的无返回值的方法
5. CONSTANT_Class：类或者接口的引用
----tag 7
----name_index u2：类或接口名，指向utf8
结构示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-09.jpg)

**3)字段，方法，接口方法的常量表示：**
1. 组成：CONSTANT_Fieldref,CONSTANT_Methodref,CONSTANT_InterfaceMethodref
	- tag 9,10,11
	- class_index u2：指向CONSTANT_class
	- name_and_typr_index u2：指向CONSTANT_NameAndType
2. CONSTANT_Fieldref结构示例如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-10.jpg)
3. CONSTANT_Methodref结构示例如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-11.jpg)

**4)类索引，父类索引和接口索引：**
1. this_class：u2，当前类
通常是一个指向常量池Class（CONSTANT_class）的索引。
2. super_class：u2,父类，超类,只能有一个
也是指向CONSTANT_class的索引
3. interfaces_count：u2,接口数量
4. inserfaces：
	- interfaces_count个
	- 每个接口都会指向CONSTANT_class

### 字段，方法
**1)字段Field：**
1. field_count：字段数量
2. fields：字段详细信息(field_info)
共field_count个field_info表
3. field_info：字段中可能有的属性
	- access_flags u2：访问修饰符
	- name_index u2：名字索引
	- descriptor_index u2：描述索引
	- attributes_count u2：属性数量
	- attribute_info：attributes_count个attribute
4. access_flags，访问标识，和类的相似：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-13.jpg)
5. name_index u2：常量池引用，标识字段的名字
6. descriptor_index：
	- 表示字段的类型
	- 使用大写字母来表示类型：
	![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-14.jpg)

**2)方法Method：**
1. method_count：方法数量
2. methods：方法详细内容
method_count个method_info
3. method_info：
	- access_flags u2：访问修饰符
	- name_index u2：方法名字
	- descriptor_index u2：方法描述
	- attributes_count u2：属性数量
	- attribute_info：属性详情
4. 方法的access_flags访问修饰符：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-15.jpg)
5. name_index u2：方法名字索引(常量池utf8)
descriptor_index u2：方法描述索引(utf8)
6. 方法描述符：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-16.jpg)
可以以很简短的方式描述一个字符

---
## 3.属性及类结构示例
类文件结构中的属性内容较多，单独整理。

**1)属性简介：**
1. 在field和method中可以有若干个attribute，类文件也有attribute,用于描述一些额外的信息。
	- attribute_name_index u2：名字，指向常量池utf8
	- attribute_length u4：属性长度,个数
	- attribute_info[attribute_length] u1：属性详细信息
2. attribute本身可以包含attribute
3. 随着JDK的发展不断有新的attribute加入
4. 常见的属性表：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-17.jpg)

**2)关于属性的详细介绍：**
1. Deprecated属性，包含如下：
	- attribute_name_index u2：索引，指向Deprecated的utf8常量
	- attribute_length u4：属性长度，恒为0
2. ConstantValue：常量属性
	- attribute_name_index u2：ConstantValue的utf8常量池索引
	- attribute_length u4：属性长度，恒为2
	- constantvalue_index u2：常量值，指向常量池，可以是Utf8,Float,Double,Int等
	- 简单例子：
	![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-18.jpg)
3. Code：稍微复杂
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-19.jpg)
Code中的异常是如何处理这个异常。
Code属性的属性主要有两个：
	- LineNumberTable
	- LinVariableTable
4. LineNumberTable：Code属性的属性
用来表示字节码和行数的关系
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-20.jpg)
5. LinVariableTable：Code属性的属性
局部变量表
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-21.jpg)
6. Exceptions属性：
表示方法抛出的异常(throws抛出，不是catch捕获)，结构如下：
	- attribute_name_index u2：名字索引
	- attribute_length u4：长度
	- number_of_exceptions u2：抛出多少个异常
	- exception_index_table[n] u2：异常表，指向常量池Class
7. Code及Exceptions的属性结构示例：
代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-22-1.jpg)
Exceptions属性结构如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-23.jpg)
Code属性结构如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-24.jpg)
8. SourceFile：描述生成Class文件的源代码文件名称
	- attribute_name_index u2：名字索引
	- attribute_length u4：长度，固定为2
	- source_file u2：指向utf8常量的索引，该文件的文件名

**3)Class文件结构示例：**
1. Class文件源码如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-25.jpg)
2. 类文件结构示意图：
类字节码文件的头信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-26.jpg)
其他相关信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-27-1.jpg)
关于异常的信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-28.jpg)
关于局部变量的信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00061JVM%E5%AD%A6%E4%B9%A04-29-1.jpg)

---
## 4.Class装载验证流程



---