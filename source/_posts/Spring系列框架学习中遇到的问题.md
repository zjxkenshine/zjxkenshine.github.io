---
title: Spring系列框架学习中遇到的问题及解决方案
date: 2018-4-14 19:26:05 
tags: 
- Spring
- Spring核心框架
categories: J2EE框架

---
## 0.问题列表
1. 使用mock进行单元测试时报错：
		java.lang.UnsupportedClassVersionError: chap1/Test01 : Unsupported major.minor version 52.0
2. Spring+Junit4单元测试时遇到的错误：
		Java.lang.ExceptionInInitializerError
		....

---
## 1.问题1-5解决
**1)使用mock进行单元测试时报错：**
1. 错误：
		Unsupported major.minor version 52.0
		...一堆没用的信息
2. 搜了一大堆网上的资料，没一个对的上的。
最后发现是maven的jdk版本问题。将maven的版本修改为jdk1.8就可以了。
52.0对应的jdk就是1.8，说明低版本不支持高版本。为了以后不报错，修改maven本地setting.xml文件配置如下：
		<profile>
		<id>jdk-1.8</id>
		<activation>
			<activeByDefault>true</activeByDefault>
			<jdk>1.8</jdk>
	    </activation>	
		<properties>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
		</properties>
		</profile>
详情参考[maven学习笔记2]()。

**2)Spring+Junit4测试时出错：**
1. 更多错误：
		java.lang.ExceptionInInitializerError
		at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
		at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
		...省略n行
		at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
		Caused by: java.lang.IllegalStateException: SpringJUnit4ClassRunner requires JUnit 4.12 or higher.
		at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.<clinit>(SpringJUnit4ClassRunner.java:102)
		... 19 more
2. 解决方案：
主要看那个course by：
		java.lang.IllegalStateException: SpringJUnit4ClassRunner requires JUnit 4.12 or higher
将pom.xml中的Junit版本改为更高的即可：
![](http://p5ki4lhmo.bkt.clouddn.com/00041Spring%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-01.jpg)



---

