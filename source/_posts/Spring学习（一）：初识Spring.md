---
title: Spring核心框架学习（一）：初识Spring及其生态圈
date: 2018-4-14 19:15:31 
tags: Spring核心框架
categories: 
- Spring
- Java框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)
参考视频：某培训班视频

2. 内容概述：
	- Spring简介
	- Spring的Bean容器
	- Spring核心模块
	- Spring生态系统


---
## 1.一些名词的解释
后面会涉及到的名词的介绍：
- `Java EE`：java的企业级应用，又称为J2EE。
Java EE的核心是EJB3.0, 其提供了更兼便捷的企业级的应用框架。
- `EJB`：是sun的JavaEE服务器端组件模型，设计目标与核心应用是部署分布式应用程序。简单来说就是把已经编写好的程序打包放在服务器上执行。
在J2EE里，Enterprise Java Beans(EJB)称为Java 企业Bean，是Java的核心代码，分别是会话Bean（Session Bean），实体Bean（Entity Bean）和消息驱动Bean（MessageDriven Bean）。在EJB3.0推出以后，实体Bean被单独分了出来，形成了新的规范JPA。
EJB 从技术上而言不是一种"产品，而是一种描述了构建应用组件要解决的标准/规范。
- `JPA`：Java Persistence API的简称，Java持久层API
- `POJO`：Plain Ordinary Java Objec简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。
- `IOC`：控制反转（Inversion of Control，英文缩写为IoC）把创建对象的权利交给框架,是框架的重要特征，并非面向对象编程的专用术语。它包括依赖注入（Dependency Injection，简称DI）和依赖查找（Dependency Lookup）。
- `DI`：依赖注入。

---
## 2.Spring简介
1. Spring是一个开源框架，最早是为了解决企业级应用开发的复杂性而创建的。使用Spring可以让简单的JavaBean实现之前只有EJB才能完成的事情。
EJB主要被用来做分布式企业级应用开发，但是Spring不具备分布式能力。
但是Spring不仅仅局限于服务器端的开发，它是一个很庞大的生态体系。涵盖软件开发的各个方面。
2. Spring降低Java开发复杂性的4个关键策略：
	- 基于POJO的轻量级和最小侵入式编程。
	- 通过依赖注入(DI)和面向接口实现松耦合。
	- 基于切面和惯例进行声明式编程。
	- 通过切面和模板减少样板式代码。
3. 下面简单介绍这四个策略。
4. 介绍之前，需要创建一个Maven项目用于理解和测试：
创建一个Maven项目，并且引入spring的依赖，pom.xml配置如下：
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>SpringTest</groupId>
		  <artifactId>spring1</artifactId>
		  <packaging>war</packaging>
		  <version>0.0.1-SNAPSHOT</version>
		  <name>spring1 Maven Webapp</name>
		  <url>http://maven.apache.org</url>
		<properties>  
			<junit.version>4.12</junit.version>  
			<spring.version>4.3.10.RELEASE</spring.version>  
			<commons-logging.version>1.2</commons-logging.version>
		</properties> 
		  <dependencies>
			<dependency>
			  <groupId>junit</groupId>
			  <artifactId>junit</artifactId>
			  <version>${junit.version}</version>
			  <scope>test</scope>
			</dependency>
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>4.0.0</version>
				<!-- 只在编译和测试时运行 -->
				<scope>provided</scope>
			</dependency>
			<!-- 添加Spring依赖的jar包-->  
			<!--依赖注入包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-aop</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!--切片包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-aspects</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!-- Beans包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-beans</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!-- 容器包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-context</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!-- 容器依赖包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-context-support</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!-- 核心包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-core</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!-- 表达式包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-expression</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!--spring-framework-bom包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-framework-bom</artifactId>  
			    <version>${spring.version}</version> 
			    <type>pom</type> 
			</dependency>  
			<!--spring-instrument包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-instrument</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--连接数据库包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-jdbc</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring消息包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-jms</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring信息包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-messaging</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring对象映射包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-orm</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--spring-oxm包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-oxm</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring测试包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-test</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring事物管理包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-tx</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--Spring文本项目包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-web</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--SpringMVC包-->  
			<dependency>  
			    <groupId>org.springframework</groupId>  
			    <artifactId>spring-webmvc</artifactId>  
			    <version>${spring.version}</version>  
			</dependency>  
			<!--spring-websocket包-->  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-websocket</artifactId>  
				<version>${spring.version}</version>  
			</dependency>  
			<!--Spring 依赖commons-logging包-->  
			<dependency>  
				<groupId>commons-logging</groupId>  
				<artifactId>commons-logging</artifactId>  
				<version>${commons-logging.version}</version>  
			</dependency>  
		</dependencies>
		<build>
			<finalName>spring1</finalName>
			<plugins>
				<plugin>
				<!--
					<groupId>org.mortbay.jetty</groupId>
					<artifactId>jetty-maven-plugin</artifactId>
					<version>8.1.16.v20140903</version>  -->
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<version>2.2</version>
					<executions>
						<execution>
							<phase>package</phase>
							<goals>
								<goal>run</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
			</plugins>
		</build>
		</project>
之后在属性中选择动态的web项目，Maven update一下，就行了。
创建好之后如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-01.jpg)

----
## 3.Spring Framwork的4个策略介绍
**1)基于POJO的轻量级和最小侵入式编程**
1. 侵入式编程：
强迫应用继承他们的类或者实现他们的接口从而导致应用与框架绑死。
2. Spring的非侵入式编程模型意味着这个类在Spring应用和非Spring应用中都可以发挥同样的作用。
3. 尽管看起来很简单，但是POJO依旧是可以实现很多功能的，实现的方式之一就是通过依赖注入DI来装配他们。
4. 就像一个简单的POJO一样：
		package chap1;
		public class HelloWorldBean {
			public String sayHello(){
				return "Hello World";
			}
		}
这就是一个普通的javaBean,看不出和Spring有任何的关联，这就是Spring想要做的。

### 依赖注入简介
1. 依赖注入一键演变成为一项复杂的编程技巧或者设计模式理念。
在项目中使用DI，代码会异常简单且易于理解和测试。
2. 传统的做法(反正我是没有这么做过)：
每个对象负责管理与自己相互协作的对象的引用，会导致高度耦合且难以写测试代码。如下面这个学生需要执行打字任务：(紧耦合的写法)
		package chap1;
		public class Student1 {
			private TypeTask task;
			public Student1(){
				this.task=new TypeTask(); //紧耦合
			}
			public void doTask(){
				task.execute();
			}
		}
打字工作类如下：
		package chap1;
		public class TypeTask implements Task{
			@Override
			public void execute(){
				System.out.println("执行了一个任务");
			}
		}
工作接口：
		package chap1;
		public interface Task {
			public void execute();
		}
这样子这个学生只能完成一种固定的任务typetask，很不灵活。
这种紧密耦合的代码难以测试，难以复用，难以理解。
但是有不能玩没有耦合，否则代码将毫无作用。
3. 通过DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象时进行设定。依赖关系将被自动注入到需要他们的对象中去：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-02.jpg)
4. 创建Student2，代码如下：
		package chap1;
		public class Student2 {
			private Task task;
			public Student2(Task task){		//构造器注入
				this.task=task;
			}
			public void doTask(){
				task.execute();
			}
		}
这是依赖注入的方式之一，构造器注入(constructor injection)。
注意，这里使用的是接口，而不是具体的实现类。
在测试时也非常的简单，更本不用考虑是否有实现类或者是什么具体工作实现，只要使用mock进行依赖替换就可以了。
5. 首先添加Mockito的pom.xml依赖：
		package chap1;
		import org.junit.Test;
		import static org.mockito.Mockito.*;
		public class Test01 {
			@Test
			public void testStudent02(){
				Task MockTask=mock(Task.class);		//创建mockTask
				Student2 student2=new Student2(MockTask);		//注入mockTask
				student2.doTask();
				verify(MockTask,times(1)).execute();	//使用Mockito框架验证MockTask的execute方法只被执行一次。
			}
		}
测试结果：（绿了就是最好的结果）
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-03.jpg)


---
## 4. Spring容器及Bean的生命周期

---
## 5. Spring生态系统简介(各种有用的工具模块)

---



---