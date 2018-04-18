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
3. 学习XML装配Bean时单元测试又报错了：
>Caused by: java.lang.IllegalStateException: Neither GenericXmlContextLoader nor AnnotationConfigContextLoader was able to load an ApplicationContext from [MergedContextConfiguration@69379752 testClass = HppyMusicTest, locations = '{}', classes = '{}', contextInitializerClasses = '[]', activeProfiles = '{}', propertySourceLocations = '{}', propertySourceProperties = '{}', contextCustomizers = set[[empty]], contextLoader = 'org.springframework.test.context.support.DelegatingSmartContextLoader', parent = [null]].
4. Spring xmlsetting方式注入测试时又出错了：
>org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'normalMusic' defined in class path resource [chap2/musicplayer3.xml]: Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [chap2.NormalMusic]: No default constructor found; nested exception is java.lang.NoSuchMethodException: chap2.NormalMusic.<init>()

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

**3）学习XML装配Bean时单元测试又报错了：**
1. 错误日志(只截取了course by那段)：
Caused by: java.lang.IllegalStateException: Neither GenericXmlContextLoader nor AnnotationConfigContextLoader was able to load an ApplicationContext from [MergedContextConfiguration@69379752 testClass = HppyMusicTest, locations = '{}', classes = '{}', contextInitializerClasses = '[]', activeProfiles = '{}', propertySourceLocations = '{}', propertySourceProperties = '{}', contextCustomizers = set[[empty]], contextLoader = 'org.springframework.test.context.support.DelegatingSmartContextLoader', parent = [null]].
2. 错误原因：
在测试类中使用了
`@RunWith(SpringJUnit4ClassRunner.class)`注解而未使用`@ContextConfiguration()注解`,而使用手动加载上下文，这样子就会出错。
3. 解决方法：
将@RunWith(SpringJUnit4ClassRunner.class)去除，不使用Spring提供的测试，手动加载xml配置文件的上下文：
		AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(MusicPlayer2Config.class);
或者将手动加载的去除，使用注解的locations属性自动加载上下文。

**4) Spring xml setting方式注入测试时又出错了：**
1. 错误代码(核心部分)：
>org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'normalMusic' defined in class path resource [chap2/musicplayer3.xml]: Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [chap2.NormalMusic]: No default constructor found; nested exception is java.lang.NoSuchMethodException: chap2.NormalMusic.<init>()
2. 错误原因：
NormalMusic没有默认的构造方法。
说明Setting注入配置的bean都需要有默认为空的构造方法。
3. 解决，添加默认的构造方法就行：
		...
		public NormalMusic(){
		}
		...

---

