---
title: MyBatis学习笔记（一）：MyBatis入门
date: 2018-3-20 22:42:27 
tags: MyBatis3
categories: Java框架

---
## 0.学习准备：
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis官方文档][1]

2. 本章重点
	- 了解Mybtis
	- 使用Maven创建一个简单的MyBatis项目
	- 配置Mybatis
	- 核心配置文件的解释

---
## 1.MyBatis简介
1. 发展：
2001年由Clinton Begin发起，原名iBatis，2004年并入Apache,2010年改名为Mybatis。发展成了一个Java的持久层框架。
2. 特点：
支持自定义SQL查询，存储过程和高级映射的存储层框架。
消除了所有JDBC代码和参数的手动设置及结果集的检索。
与其他ORM(对象关联映射)的框架不同，Mybatis没有将Java对象与数据库表关联，而是将SQL语句与Java方法关联。
ORM:Object Relation Mapping
3. 缓存：
MyBatis支持声明式数据缓存。
MyBatis默认提供了基于Java HashMap的缓存实现，以及用于OSCache,Ehcache,Hazelcast和Memchached连接的默认连接器。
4. MyBatis官方GitHub地址：
<https://github.com/MyBatis>
官方文档：
<http://www.mybatis.org/mybatis-3/zh/index.html>


---
## 2.使用Maven创建一个项目：
**1)创建项目：**
1. 在Eclipse中点击[File]-->[New]-->[Other]-->[Maven Project]:
![](http://p5ki4lhmo.bkt.clouddn.com/00018MyBatis%E5%AD%A6%E4%B9%A01-01.jpg)
勾选第一项，点击next。
2. 输入组名，模块名，版本号：
![](http://p5ki4lhmo.bkt.clouddn.com/00018MyBatis%E5%AD%A6%E4%B9%A01-02.jpg)
点击finish创建完毕。

**2)pom.xml基本配置：**
1. 目前的pom.xml:
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>Mabatis</groupId>
		  <artifactId>simple</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		</project>
2. 添加UTF-8编码：
		  <properties>
		  	<project.build.sourceEncoding> UTF-8</project.build.sourceEncoding>
		  </properties>
3. 设置编译源代码的JDK版本(1.7)：
		  <build>
		  	<plugins>
		  		<plugin>
		  			<artifactId>maven-compiler-plugin</artifactId>
		  			<configuration>
		  				<source>1.7</source>
		  				<target>1.7</target>
		  			</configuration>
		  		</plugin>
		  	</plugins>
		  </build>

**3)其他配置：**
1. MyBatis依赖：
		<dependencies>
		  	<dependency>
		  		<groupId>org.mybatis</groupId>
		  		<artifactId>mybatis</artifactId>
		  		<version>3.3.0</version>
		  	</dependency>
		  </dependencies>
2. JUnit依赖：
		<dependency>
		  	<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
3. Log4j依赖:
	  	<dependency>
	  	  	<groupId>org.slf4j</groupId>
	  		<artifactId>slf4j-api</artifactId>
	  		<version>1.7.12</version>
	  	</dependency>
	  	  	<dependency>
	  	  	<groupId>org.slf4j</groupId>
	  		<artifactId>slf4j-log4j12</artifactId>
	  		<version>1.7.12</version>
	  	</dependency>
	  	<dependency>
	  	  	<groupId>log4j</groupId>
	  		<artifactId>log4j</artifactId>
	  		<version>1.2.17</version>
	  	</dependency>
4. Mysql驱动的依赖：
	  	<dependency>
	  	  	<groupId>mysql</groupId>
	  		<artifactId>mysql-connector-java</artifactId>
	  		<version>5.1.38</version>
	  	</dependency>
5. 完整的pom.xml配置：
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>Mabatis</groupId>
		  <artifactId>simple</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  
		  <properties>
		  	<project.build.sourceEncoding> 
		  	UTF-8
		  	</project.build.sourceEncoding>
		  </properties>
		  
		  <dependencies>
		  
		  	<dependency>
		  		<groupId>org.mybatis</groupId>
		  		<artifactId>mybatis</artifactId>
		  		<version>3.3.0</version>
		  	</dependency>
		  	<dependency>
		  	  	<groupId>junit</groupId>
		  		<artifactId>junit</artifactId>
		  		<version>4.12</version>
		  		<scope>test</scope>
		  	</dependency>
		  	<dependency>
		  	  	<groupId>org.slf4j</groupId>
		  		<artifactId>slf4j-api</artifactId>
		  		<version>1.7.12</version>
		  	</dependency>
		  	  	<dependency>
		  	  	<groupId>org.slf4j</groupId>
		  		<artifactId>slf4j-log4j12</artifactId>
		  		<version>1.7.12</version>
		  	</dependency>
		  	<dependency>
		  	  	<groupId>log4j</groupId>
		  		<artifactId>log4j</artifactId>
		  		<version>1.2.17</version>
		  	</dependency>
		  	<dependency>
		  	  	<groupId>mysql</groupId>
		  		<artifactId>mysql-connector-java</artifactId>
		  		<version>5.1.38</version>
		  	</dependency>
		  </dependencies>
		  
		  <build>
		  	<plugins>
		  		<plugin>
		  			<artifactId>maven-compiler-plugin</artifactId>
		  			<configuration>
		  				<source>1.7</source>
		  				<target>1.7</target>
		  			</configuration>
		  		</plugin>
		  	</plugins>
		  </build>
		</project>

---
## 3.准备数据库及配置MyBatis
**1)准备数据库:**
1. 创建数据库mybatis:
		CREATE DATABASE mybatis DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
2. 创建表：
		CREATE TABLE country(
		id TINYINT UNSIGNED AUTO_INCREMENT,
		countryname VARCHAR(255) NULL,
		countrycode VARCHAR(255) NULL,
		PRIMARY KEY(id)
		)CHARSET utf8;
3. 插入数据：
		INSERT INTO country
		VALUES(NULL,'中国','CN'),(NULL,'美国','US'),(NULL,'俄罗斯','RU'),
		(NULL,'英国','GB'),(NULL,'法国','FR');

**2)配置MyBatis*（XML方式）:**
1. 配置MyBatis的方式有很多种，其中最基础最常见的就是XML方式，还有Spring Bean方式（整合时）以及通过Java编码方式进行配置。
2. 在srv/main/resources下面创建mybatis-config.xml(约定的名字)，创建好后右键-->【Open With】-->【Xml Editor】，输入如下内容：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE configuration
		  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-config.dtd">
		  
		  <configuration>
			<settings>
				<setting name="logImpl" value="LOG4J"/>
			</settings>
			
			<typeAliases>
				<package name="Mybatis.simple.model"/>
			</typeAliases>
			
			<!-- 配置环境 -->
			<environments default="development">
				<environment id="development">
					<!-- 事务 -->
					<transactionManager type="JDBC">
						<property name="" value=""/>
					</transactionManager>
					<!-- 数据源 -->
					<dataSource type="UNPOOLED">
						<property name="driver" value="com.mysql.jdbc.Driver"/>
						<property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
						<property name="username" value="root"/>
						<property name="password" value=""/>
					</dataSource>
				</environment>
			</environments>
			
			<!-- 映射 -->
			<mappers>
				<mapper resource="Mybatis/simple/mapper/CountryMapper.xml"/>
			</mappers>
		  </configuration>

**3)以上配置的简单介绍：**
1. `<!DOCTPE>`声明这是一个MyBatis核心配置文件：
		<!DOCTYPE configuration
		  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-config.dtd">
可在[MyBatis官方文档][1]-->【入门】中找到。
定义了一个MyBatis核心配置文件，并且会出现标签补齐。
若断网无法提示可以手动下载一个`mybatis-3-config.dtd`进行如下配置：(maven仓库中就有)
点击【Windows】-->【Preferences】-->【XML-XML catalog】-->【add】-->【Catalog Entry】并进行如下配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00018MyBatis%E5%AD%A6%E4%B9%A01-03.jpg)
然后点【OK】就可以了
2. `<settings>`配置LOG4J输出日志：
		<settings>
			<setting name="logImpl" value="LOG4J"/>
		</settings>
指定使用LOG4J来输出日志。
3. `<typeAliases>`定义包的别名：
		<typeAliases>
			<package name="Mybatis.simple.model"/>
		</typeAliases>
MyBatis中会经常使用类的全限定名称，这样定义了包的别名，使用Country就等于是使用"Mabatis.simple.model.Country"。暂未创建。
4. 配置环境：
		<environments default="development">
			<environment id="development">
				<!-- 事务 -->
				<transactionManager type="JDBC">
					<property name="" value=""/>
				</transactionManager>
				<!-- 数据源 -->
				<dataSource type="UNPOOLED">
					<property name="driver" value="com.mysql.jdbc.Driver"/>
					<property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
					<property name="username" value="root"/>
					<property name="password" value=""/>
				</dataSource>
			</environment>
		</environments>
可以配置多个环境，默认使用id为development的环境。
事务`<transactionManager>`的type一般都指定为JDBC。
数据源`<dataSource>`则是JDBC中连接数据库的操作。
5. `<mapper>`配置映射：
		<mappers>
			<mapper resource="Mybatis/simple/mapper/CountryMapper.xml"/>
		</mappers>
CountryMapper.xml是MyBatis的SQL语句及映射文件，暂时还未创建该映射。

---
## 4.创建实体类及映射文件：
**1)创建实体类：**
1. 实际应用中一个表一般都会对应一个实体，用于增删改及简单的查操作。
2. 在src/main/java创建一个Mybatis.simple.model包，并创建实体类Country如下:
		package Mybatis.simple.model;
		
		public class Country {
			private int id;
			private String countryname;
			private String countrycode;
			//get,set及toString方法
		}

**2)创建映射文件(Mapper.xml)：**
1. 命名方式：
Mybatis一般习惯性的用Mapper作为映射文件和接口类名的后缀，最好这样写。
所以一般称XML为Mapper.xml文件，称接口为Mapper接口。
2. 在src/main/resources下创建MyBatis.simple.mapper目录，再在该目录下创建country映射文件（CountryMapper.xml），内容如下：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		<mapper namespace="Mybatis.simple.mapper.CountryMapper">
		
		<!-- Country使用别名 -->
		<select id="selectAll" resultType="Country">
			select * from country
		</select>
		
		</mapper>

**3)上述映射文件的简单介绍：**
1. <!DOCTYPE>定义文件类型：映射文件
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
这段内容可在[MyBatis官方文档][1]-->【入门】中找到。
离线提示的方法同核心配置文件的<!DOCTYPE>。
2. `<mapper>`是整个XML文件的根元素。
其中的namespace属性指的是这个映射文件的工作空间（即ID）,**整个工程内不能重复**。
3. `<select>`就是定义了一个select查询语句，id是该查询语句的id，在该映射内不能重复。然后就可以通过namespace.id准确定位该条SQL语句了。
resultType属性定义了当前查询的返回值，这里就是实体类，使用了核心配置文件中的别名，不然就要一大段。
4. resultType也能返回其他java基本类型，MyBatis已经定义好了一套别名，就不用写类的全路径了（如：java.lang.Math）：
具体内容可以在[MyBatis官方文档][1]-->【XML配置】-->【类型别名】中找到。

---
## 5.配置LOG4J日志：
**1)配置log4j.properties**
1. Mybatis 的内置日志工厂提供日志功能，内置日志工厂将日志交给以下其中一种工具作代理：
		SLF4J
		Apache Commons Logging
		Log4j 2
		Log4j
		JDK logging
一般使用`Log4j`就可以了。
有个Logback，是升级版的Log4j。
2. 在src/main/resources中添加log4j.properties配置文件
		# 全局配置
		log4j.rootLogger=ERROR, stdout
		# MyBatis 日志配置.
		log4j.logger.MyBatis.simple.mapper=TRACE
		# 控制态输出日志
		log4j.appender.stdout=org.apache.log4j.ConsoleAppender
		log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
		log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
只要将官方文档日志中的配置粘过来并修改MyBatis 日志配置就行。
将log4j.logger后的路径改为映射文件包的包名。

**2)书上的一些log4j简单介绍：**
1. `log4j.logger.MyBatis.simple.mapper`原本指向的是mapper包，但是可能是使用Maven的原因，创建的包会自动变为目录，所以也就指向mapper目录了。
2. log4j中所谓的包名实际只是XML配置中namespace的一部分而已。
3. **TRACE是MyBatis日志的最低级别**，在这个级别下会输出SQL过程中的详细信息，非常适合在开发中使用。

---
## 6.创建测试类并测试：
**1)创建测试类：**
1. 在src/test/java中创建Mybatis.simple.model包，并在该包下创建CountryMapperTest.java测试类，内容如下：
		package Mybatis.simple.mapper;
		
		import java.io.IOException;
		import java.io.Reader;
		import java.util.List;
		
		import org.apache.ibatis.io.Resources;
		import org.apache.ibatis.session.SqlSession;
		import org.apache.ibatis.session.SqlSessionFactory;
		import org.apache.ibatis.session.SqlSessionFactoryBuilder;
		import org.junit.BeforeClass;
		import org.junit.Test;
		
		import Mybatis.simple.model.Country;
		
		public class CountryMapperTest {
			//SqlSession工厂类
			private static SqlSessionFactory sqlSessionFactory;
			
			@BeforeClass
			public static void init(){
				try{
					// 加载核心配置文件
					Reader reader=Resources.getResourceAsReader("mybatis-config.xml");
					//新建工厂对象，相当于JDBCconnect
					sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);
					//关闭输入流
					reader.close();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
			
			@Test
			public void testSelectAll(){
				//使用sqlSession实例化一个sqlSession对象,相当于使用预处理语句
				SqlSession sqlSession=sqlSessionFactory.openSession();
				try{
					List<Country> countryList=sqlSession.selectList("Mybatis.simple.mapper.CountryMapper.selectAll");
					for(Country coun:countryList){
						System.out.println(coun);
					}
				}finally{
					//关闭sqlSession
					sqlSession.close();
				}
			}
			
		}



**2)关于上述测试类的说明：**
1. 加载核心配置文件：
`Reader reader=Resources.getResourceAsReader("mybatis-config.xml");`
通过Resources工具将`mybatis-config.xml`核心配置文件读入Reader。
2. 新建工厂对象：
`sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);`
通过`SqlSessionFactoryBuilder`工厂建造类使用`Reader`新建一个`sqlSessionFactory`对象。
3. 实例化一个sqlSession对象：
`SqlSession sqlSession=sqlSessionFactory.openSession();`
通过openSession方法创建一个sqlSession对象。
4. 进行查询：
`sqlSession.selectList("Mybatis.simple.mapper.CountryMapper.selectAll");`
使用namespace.id(即Mybatis.simple.mapper.CountryMapper.selectAll)定位查询语句并执行。
返回值是Country的List,说明返回值是List时映射文件内的resultType要写List里面的值。
5. 一定要关闭Read及sqlSession.
6. 简单的描述：
获取配置文件的read对象
创建sqlSessionFactory工厂对象
获取sqlSession对象
使用sqlSession的方法进行查询
关闭sqlSession对象

**3)运行测试：**
1. 两种运行方式：
	- 项目-->右键-->【RUN AS】-->【Maven Bulider】-->输入test
	- CountryMapperTest.java-->右键-->【RUN AS】-->【Junit Test】
2. 测试结果：
Junit未报错且控制台输出如下信息，成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00018MyBatis%E5%AD%A6%E4%B9%A01-04.jpg)
若测试失败，可以查看配置文件是否正确。
但是，说好的查询语句呢?怎么没有?
3. 修改映射文件CountryMapper.xml：
将`select * from Country`改为：
		select id,countryname,countrycode from country
再次运行，然后SQL语句就输出了。。。，很奇怪：
![](http://p5ki4lhmo.bkt.clouddn.com/00018MyBatis%E5%AD%A6%E4%B9%A01-05.jpg)

---
[1]:http://www.mybatis.org/mybatis-3/zh/index.html