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
- `DI`：Dependency Injection，依赖注入。
- `DL`：Dependency Lookup，依赖查找
- `AOP`:aspect-oriented programing,面向切面的编程
- `JMS`：JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。
- `JNDI`：JNDI(Java Naming and Directory Interface,Java命名和目录接口)是SUN公司提供的一种标准的Java命名系统接口，JNDI提供统一的客户端API，通过不同的访问提供者接口JNDI服务供应接口(SPI)的实现，由管理者将JNDI API映射为特定的命名服务和目录系统，使得Java应用程序可以和这些命名服务和目录服务之间进行交互.

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
**1)简介及传统做法**
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

**2)构造器注入：**
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

**3)装配Bean：**
1. 这样就实现注入了，现在Student2可以传入任何Task的实现类对象。
假设新建了一个写字任务（WriteTask）,如何将这个任务传递给Student2呢。
这种创建应用组件之间协作的行为称为装配Bean。
Spring有多种装配Bean的方式，如xml,java代码装配等。
2. 创建WriteTask类：
		package chap1;
		import java.io.PrintStream;
		public class WriteTask implements Task{
			private PrintStream printStream;
			public WriteTask(PrintStream printStream){
				this.printStream=printStream;
			}
			@Override
			public void execute() {
				printStream.println("do write task");
			}
		}
这里的printStream.println()就是基于io流的输出语句。
3. 使用xml方式将Student2，writeTask以及PrintStream装配到一起。
创建student2.xml(在test/resources目录),内容如下：
		<?xml version="1.0" encoding="UTF-8"?>  
		<beans
			xmlns="http://www.springframework.org/schema/beans"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:p="http://www.springframework.org/schema/p"
			xmlns:context="http://www.springframework.org/schema/context"
			xmlns:aop="http://www.springframework.org/schema/aop"
			xsi:schemaLocation="http://www.springframework.org/schema/beans
								http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
								http://www.springframework.org/schema/aop
								http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
								http://www.springframework.org/schema/context
								http://www.springframework.org/schema/context/spring-context-2.5.xsd">
			<!--进行装配Bean--> 
			<bean id="student2" class="chap1.Student2">
				<constructor-arg ref="task1"></constructor-arg>
			</bean>  
			<bean id="task1" class="chap1.WriteTask">
				<constructor-arg value="#{T(System).out}"></constructor-arg>
			</bean> 
		</beans>
关于提示插件的配置可以参考博客：
[Eclipse编辑Spring配置文件xml时自动提示类class包名](https://blog.csdn.net/hh775313602/article/details/70176531) 
这里的Student2和WriteTask被声明为Spring的bean,使用了constructor-arg标签的ref属性进行依赖注入。
而task1的声明使用了Spring的表达式语言SPEL(Spring Express Language)，将System.out传入到SlayDragonQues的构造器中。
4. java语言描述的装配方式：
		@Configuration
		public class Student2Config {
			@Bean
			public Student2 student2(){
				return new Student2(task());
			}
			@Bean
			public Task task(){
				return new WriteTask(System.out);
			}
		}
可以将前面的学生类都实现一个Student接口，那么这里的student bean就可以这样写：
		@Bean
		public Student Student(){
			return new Student2(task());
		}
这样子student只知道自己依赖于Task，但是并不知道这个task来自哪里。
只有通过他的配置才能了解这些部分是如何装配的。

**4)应用上下文(Application Context)的使用：**
1. Spring通过应用上下文装载bean的定义并将他们组装起来。**应用上下文全权负责对象的创建和组装**。Spring自带了多种应用上下文的实现，他们的区别仅仅是如何加载配置。
2. 以xml为例，选择ClassPathXmlApplicationContext加载xml,并获得student2对象。测试代码如下：
		@Test
		public void testStudent2Context(){
			//加载Spring上下文
			ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("chap1/student2.xml");
			//获取id为student2的Bean
			Student2 student2=context.getBean(Student2.class);
			//测试方法
			student2.doTask();
			//关闭上下文
			context.close();
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-04.jpg)
而这个student2则完全不知道自己执行了那种类型的工作。除非去看xml配置文件。
3. 根据我的理解所画的实现图(xml方式)：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-05.jpg)

### AOP面向切面编程简介
**1)简介：**
1. DI能够让相互协作的软件组件保持松耦合，而面向切面编程AOP则可以将遍布应用各处的组件分离出来形成可重用的组件。
2. 在系统中经常会有一些功能贯穿各个组件，如日志，事务管理及系统安全等系统服务，这些系统服务被称为**横切关注点**，会跨越系统的多个组件。
3. 如果将关注点分散到各个组件中，那么每个组件都要考虑这些，组件会因为与自身核心业务无关的代码导致混乱，也会给代码带来双重的复杂性，变的难以维护与测试。如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-06.jpg)
4. 而AOP则能使这些服务模块化，并**以声明的方式**将它们应用到需要使用这些服务的组件中去。能够确保POJO的简单性。
可以将切面想象成覆盖在很多组件之上的外壳，如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-07.jpg)

**2)没有使用Aop的情况：**
1. 假设有一个老师来监督每个学生的任务，在每次学生任务开始之前与任务结束都有相关的操作，创建教师类如下：
		package chap1;
		import java.io.PrintStream;
		public class Teacher {
			private PrintStream printStream;
			public Teacher(PrintStream printStream){
				this.printStream=printStream;
			}
			public void doBeforeTask(){
				printStream.println("now studend begin the task");
			}
			public void doAfterTask(){
				printStream.println("now studend finish the task");
			}
		}
2. 创建student3如下：
		package chap1;
		public class Student3 {
			private Task task;
			private Teacher teacher;
			public Student3(Task task,Teacher teacher){
				this.teacher=teacher;
				this.task=task;
			}
			public void doTask(){
				teacher.doBeforeTask();
				task.execute();
				teacher.doAfterTask();
			}
		}
这样可以实现效果，但是发现老师的对象竟然交给学生对象去管理，这就很不和逻辑了。而且有时也不一定有老师监督，所以是否还需要一个没老师的Student3?这样就会使代码变得非常复杂。
3. 而使用AOP，则可以解决这类问题。

**3)AOP的使用：**
1. 修改Student2.xml文件，给Teacher配置一个Bean并声明为切面类型：
		<?xml version="1.0" encoding="UTF-8"?>  
		<beans ...>
			<!-- 其他bean配置 -->
			<!-- 声明teacher bean -->
			<bean id="teacher" class="chap1.Teacher">
				<constructor-arg value="#{T(System).out}"></constructor-arg>
			</bean> 
			<aop:config>
				<aop:aspect ref="teacher">
				<!-- 声明切点：doTask方法 -->
				<aop:pointcut expression="execution(* *.doTask(..))" id="dotask"/>
				<!-- 声明前置通知(切点前) -->
				<aop:before pointcut-ref="dotask" method="doBeforeTask"/>
				<!-- 声明后置通知(切点后) -->
				<aop:after pointcut-ref="dotask" method="doAfterTask"/>
				</aop:aspect>
			</aop:config>
		</beans> 
2.	测试代码和上面的项同，测试结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-08.jpg)
3. 可以看到这样的写法和未使用切面的效果是一样的。
上述配置中使用aop系列标签将teacher声明为一个切面，并定义了一个切点及在这个切点附近的方法。
切点定义时的expression属性使用的是AspectJ的切点表达式语句。
这样配置之后有以下优点：
	- 首先Teacher类代码仍是一个简单的POJO，没有任何地方表名它与AOP有关，但实际上在上下文中它就已经变成一个切面了。
	- 尽管Teacher变成了一个切面，但是其他spring bean能做的事它也同样能做。
4. 我理解的Aop的实现过程：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-09.jpg)

### 模板消除样板式代码
1. 有时候写一些代码，为了实现简单的或者通用的业务，不得不一遍又一遍编写简单的代码。如JDBC,JMS,JNDI和使用REST服务时。
2. Spring旨在通过模板封装来消除样板式代码。
3. 如下列查询方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-10.jpg)
就使用了JDBC模板jdbcTemplate模板，从而不用再编写那些重复的JDBC语句了。
4. 上述的方法的queryForObject是一个接收三个参数的方法：SQL语句，查询对象以及查询条件。

---
## 4. Spring容器及Bean的生命周期
**1)Bean容器**
1. 在基于Spring的应用中，应用对象生存在Spring的容器中。Spring容器负责创建，装配，配置并并管理它们的整个生命周期。
2. 容器是Spring的核心。Spring容器使用DI管理构成应用的组件，会创建相互协作的组件之间的关联。这些对象简单干净，更有易于理解，更易于重用。而且更方便单元测试。
3. SprIng的容器有两种不同类型，一种是最最简单的Bean工厂，另一种则是基于Bean工厂创建的应用上下文。
Bean工厂提供最基本的DI支持，但是对于应用来说往往太过低级了。而应用上下文则提供了架构级别的服务，所以一般都是使用应用上下文的。

**2）应用上下文：**
1. Spring自带了很多种应用上下文，常见的有这5种：
`AnnotationConfigApplicationContext`：从一个或多个基于java的配置类中加载Spring上下文。
`AnnotationConfigWebApplicationContext`：从一个或多个基于java的配置类中加载Spring Web上下文。
`ClassPathXmlApplicationcontext`：从类路径下的一个或者多个XML配置文件中加载上下文。
`FileSystemXmlapplicationcontext`：从文件系统下的一个或者多个xml配置文件中加载上下文。
`XmlWebApplicationContext`：从Web文件下的一个或者多个XML配置文件中加载上下文定义。
2. 其他的后面都会涉及，暂时只介绍`FileSystemXmlapplicationcontext`，`ClassPathXmlApplicationcontext`以及`AnnotationConfigApplicationContext`
3. `FileSystemXmlapplicationcontext`使用方式：
		ApplicationContext context=new FileSystemXmlApplicationContext("c:/student2.xml");
`ClassPathXmlApplicationcontext`使用方式：
		ApplicationContext context=new FileSystemXmlApplicationContext("student2.xml");
这两者的区别是前者会在指定的文件系统路径下去寻找这个文件，而后者则是在所有的类路径下去寻找（包括jar文件）
4. 从java配置中加载上下文：
		ApplicationContext context=new AnnotationConfigApplicationContext("c:/student2.xml");

### Bean的生命周期
**1)简单示意图：**
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-11.jpg)
**2)生命周期简介：**
1. Spring对bean实例化
2. Spring将值和bean引用注入到bean对应的属性中
3. （如果bean实现了BeanNameWare接口），则将Bean的ID传给setBeanName方法
4. （如果bean实现了BeanFactoryAware接口），则会调用setBeanFactory方法，将BeanFactory容器实例传入
5. （如果bean实现了ApplicationContextAware接口），则会调用setApplicationContext()方法，将bean所在的应用上下文传进来
6. （如果Bean实现了BeanPostProcessor接口），则调用postPeocessBeforeInitializACTION()方法
7. （如果Bean实现了InitializingBean接口），则调用afterProperitiesSet（）方法，若使用init-method声明了初始化方法，则该方法也会被调用。
8. （如果Bean实现了BeanPostProcessor接口），则调用则调用postPeocessAfterInitializACTION()方法
9. 此时Bean已经可以开始使用了，它们会停留在应用上下文中直到应用上下文被销毁。
10. （如果Bean实现了DisposableBean接口），则会调用destory()接口方法来销毁，如果bean使用了destory-method来声明销毁方法，该方法也会被调用。


---
## 5. Spring生态系统简介(各种有用的工具模块)
**1)关于Spring系列工具：**
1. Spring为开发者提供了一个一站式的轻量级应用开发平台,提供给开发者多种的技术选择。当然Spring还有许多值得注意的子项目,了解这些子项目,可以更好地使用Spirng或理解其设计架构和思想。
2. Spring核心框架简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00039Spring%E5%AD%A6%E4%B9%A01-12.jpg)
3. 核心框架的介绍：
	- **Spring核心容器**：管理Bean的创建配置和管理，包含了bea工厂(提供DI功能)和应用上下文的实现，还提供了很多企业级服务如Email等。所有的Spring模块都是构建在核心容器之上的。
	- **Spring AOP模块**：提供了面向切面的支持，可以将遍布系统的服务解耦。
	- **数据访问与集成**：提供了JDBC数据访问的支持，提供了JMS消息处理的支持，而且会调用aop模块为Spring中的对象提供事务支持。
	- **Web与远程连接**：Spring MVC框架，还提供了多种与其他应用交互的远程调用方案。自带有远程调用框架：HTTP invoker。还提供可暴露和REST API的支持。
	- **Instrumentation**：提供了为JVM添加代理的功能，使用场景非常有限。
	- **Test测试**:Spring为使用JNDI,Servlet和Portlet编写单元测试提供了单元测试，还有继承测试的支持。

**2）常用的功能，框架：**
>Spring Framework(Core)： Spring的核心项目,其中包含了一系列的IOC容器的设计，提供了依赖注入的实现；同时,还集成了AOP,提供了面向切面编程的实现;当然还有MVC、JDBC、事务处理模块的实现。目前官网最高版本4.3.0

>Spring Boot :提供了快速构建Spring应用,提供开发效率,达到 开箱即用---- 快速开始需求开发而不被其他方面影响 “即时运行”。

>Spring Batch:提供构建批处理应用和自动化操作的框架，专门用于离线分析程序,数据批处理等场景。

>Spring Data:提供使用非关系型数据的能力,比如当基础数据并非存储在关系数据库中,或MapReduce中的分布式存储、云计算存储环境等 

>Spring Security:用户认证、授权、安全服务等工具,最先前在Spring社区中的名字是Acegi框架。

>Spring Security OAuth:OAuth是一个第三方的模块,提供一个开放的协议的实现,通过这个协议前端桌面应用可以对web应用进行简单而标准的安全调用

>Spring Web Flow:Web工作流引擎,定义了一种特定的语言来描述工作流,同时高级的工作流控制器引擎可以管理会话状态。

>Spring BlazeDS Integration :提供Spring与Adobe Flex技术集成的模块。

>Spring Dynamic Modules:提供Spring 应用运行在OSGi平台上 OSGi面向java的动态模型系统,Eclipse就是构建在OSGi平台上的。

>Spring Intergration:为企业的数据集成提供了解决方案,

>Spring AMQP:高级消息队列协议,支持java 和.NET两个版本。SpringSoruce旗下的Rabbit MQ就是一个开源的AMQP的消息服务器,Rabbit MQ 是用Erlang语言开发的。

>Spring .NET：为.NET提供Spring相关的技术支持,如IOC容器、AOP等。

>Spring Android:为Android终端开发应用提供Sring支持。

>Spring Mobile:为移动终端的服务器应用开发提供支持。

>Spring Social：Spring框架的扩展,提供了SNS服务,如FaceBook和Twitter服务。

---
