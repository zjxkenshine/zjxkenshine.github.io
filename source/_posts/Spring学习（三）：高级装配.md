---
title: Spring学习（三）：高级装配
date: 2018-04-19 10:15:52
- Spring
- Spring核心框架
categories: J2EE框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)

2. 内容概述：
	- Spring profile
	- 条件化的Bean声明
	- 自动化装配与歧义性
	- bean的作用域
	- SPEL(Spring Express Language)表达式语言

---
## 1.学习中出现的一些新概念
1. 嵌入式数据库：
这种数据库嵌入到了应用程序进程中，消除了与客户机服务器配置相关的开销。嵌入式数据库实际上是轻量级的，在运行时，它们需要较少的内存。它们是使用精简代码编写的，对于嵌入式设备，其速度更快，效果更理想。如SQLite，Empress,OpenBaseLite等。
2. `JNDI`:Java API命名与目录接口。
jdni是一种将对象和名字绑定的技术，容器产生对象并都和唯一的名字绑定，这样外部程序就用JNDI技术通过名字来获取对象，跟反射一样。(网上大佬)
jdbc是java去找数据库驱动，jndi是通过你的服务器配置（如Tomcat）的配置文件context来找数据库驱动~
3. `QA`：质量检测。
4. `javax`：java extension,java扩展api接口。而java.XXX则是标准的API接口。

---
## 2.环境迁移时的问题（嵌入式数据库）
**1)环境迁移的场景**
1. 将应用程序从一个环境迁移到另一个环境，很多时候即便是迁移成功了也无法正常工作，因为数据库配置，加密算法以及与外部系统集成都是会改变的因素。
2. 假设需要使用的是Hypersonic嵌入式数据库：
需要使用到javax.sql.DataSource这个接口，但是我用的java(8)已经自带了，所以就不用导jar包了。其实作用和驱动包相同。
需要使用到Spring框架中的EmbeddedDatabaseBuilder方法，该方法用于搭建一个Hypersonic数据库，使用该类的build()方法会返回一个连接实例。(具体使用见后面)
3. 关于Javax.sql.DataSource的用法可以参考以下博客：
<https://blog.csdn.net/guohuiJI/article/details/72458016>

**2)简单使用嵌入式数据库：**
1. JavaConfig配置类代码如下：
		package chap3;
		import javax.sql.DataSource;
		import org.springframework.context.annotation.Bean;
		import org.springframework.context.annotation.Configuration;
		import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
		@Configuration
		public class SqlConfig {
			@Bean
			private DataSource dataSource(){
				return new EmbeddedDatabaseBuilder()
				.addScript("classpath:schema.sql")
				.addScript("classpath:test-data.sql")
				.build();
			}
		}
Hypersonic数据库创建的数据库的模式定义在schema.sql中，测试数据则是通过test-data.sql加载的。
在开发环境中运行集成测试或者启动应用进行手动测试的时候，这样创建DataSource是非常有用的。每次启动它的时候都能让数据库处于一个给定(开启服务)的状态。
2. 尽管只使用EmbeddedDatabaseBuilder创建的DataSource非常实用与开发环境，但是对于环境却并不是那么好用。所以一般使用JNDI容器获取的DataSoutce，但是对于简单的开发测试环境JNDI会带来不必要的复杂性：
		@Bean
		private DataSource dataSource2(){
			//使用jndi从容器获取
			JndiObjectFactoryBean jndiObjectFactoryBean=new JndiObjectFactoryBean();
			jndiObjectFactoryBean.setJndiName("jdbc/kenshine");
			jndiObjectFactoryBean.setResourceRef(true);
			jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
			return (DataSource) jndiObjectFactoryBean.getObject();
		}
JndiObjectFactoryBean用于配置数据源，至于这个JndiName我是乱编的，需要配置的话要到Tomcat服务器的Context.xml中配置数据库的连接属性。


---

