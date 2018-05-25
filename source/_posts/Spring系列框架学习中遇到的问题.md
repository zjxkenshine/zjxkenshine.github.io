---
title: Spring系列框架学习中遇到的问题及解决方案
date: 2018-4-14 19:26:05 
tags: 
- Spring
- Spring核心框架
categories: Java框架

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
5. 创建自定义的properties文件时报错：
		Contens is not allowed prolog
6. 学习Spring AOP时出错：
		warning no match for this type name: name [Xlint:invalidAbsoluteTypeName]
7. SpringMVC配置web.xml后启动Tomcat报错：
		java.lang.ClassNotFoundException: org.springframework.web.servlet.DispatcherServlet
8. 使用Spring表单标签库时出错：
		The absolute uri: http://www.springframework.org/tags/form cannot be resolved in either web.xml or the jar files deployed with this application

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

**5) 创建自定义的properties文件时报错：**
1. 详细错误：
		Contens is not allowed prolog
2. 出错原因，一般是.xml文件强转为.properties时的格式问题。
3. 解决方案
将格式转换为UTF-8,等一段时间就好了。

---
## 2.问题6~10的解决方案
**6)学习Spring AOP时出错**
1. 错误简介：
		warning no match for this type name: name [Xlint:invalidAbsoluteTypeName]
2. 出错代码定位：
		@Aspect
		@Component
		public class DanceMoniter {
			@Pointcut("execution(** chap4.Performance2.perform(String) and args(name)")
			public void perform(String name){}
			
			@Around("perform(name)")
			public void AroundPerform(ProceedingJoinPoint joinPoint){
				try{
					System.out.println("performance begin!");
					joinPoint.proceed();
					System.out.println("performance end!");
					System.out.println("the performance is great!");
				}catch(Throwable e){
					System.out.println("the performance is awful!");
				}
			}
		}
3. 原因：
Pointcut中语法错误。
proceed没有参数。(参数不匹配)
4. 修改如下：
		@Aspect
		@Component
		public class DanceMoniter {
			@Pointcut("execution(** chap4.Performance2.perform(String)） and args(name)")
			public void perform(String name){}
			
			@Around("perform(name)")
			public void AroundPerform(ProceedingJoinPoint joinPoint,String name){
				try{
					System.out.println("performance begin!");
					joinPoint.proceed(new String[]{name});	//主要是这里
					System.out.println("performance end!");
					System.out.println("the performance is great!");
				}catch(Throwable e){
					System.out.println("the performance is awful!");
				}
			}
		}
终于解决了。如果使用@Before等则不会出现这个问题。

**7)SpringMVC配置web.xml后启动报错：**
1. 完整错误：
		严重: Servlet [springDispatcherServlet] in web application [/SpringMVC-1] threw load() exception
		java.lang.ClassNotFoundException: org.springframework.web.servlet.DispatcherServlet
		at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1333)
		at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1167)
		at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:518)
		at 
		...
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
		at java.lang.Thread.run(Unknown Source)
2. 出错原因：
/WEB-INF下无法访问外部的jar包
3. 解决：
项目右击-->properties-->Deployment Assembly-->add-->Java Build Path Entries-->导入所有依赖的Jar包，重新start tomcat即可。
![](http://p5ki4lhmo.bkt.clouddn.com/00041Spring%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-02.jpg)

**8)使用Spring表单标签库时报错：**
1. 报错信息：
		org.apache.jasper.JasperException: The absolute uri: http://www.springframework.org/tags/form cannot be resolved in either web.xml or the jar files deployed with this application
			org.apache.jasper.compiler.DefaultErrorHandler.jspError(DefaultErrorHandler.java:55)
			org.apache.jasper.compiler.ErrorDispatcher.dispatch(ErrorDispatcher.java:277)
			org.apache.jasper.compiler.ErrorDispatcher.jspError(ErrorDispatcher.java:75)
			......(省略n多行)
2. 出错原因：未知
3. 解决方法一：
在Tomcate部署设置中将`Serve modules without publishing`选项去除。
![](http://p5ki4lhmo.bkt.clouddn.com/00041Spring%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-03.jpg)
该选项可以让tomcat直接使用WebContent下的东西。
4. 解决方法二：详见博客
<https://blog.csdn.net/flashdelover/article/details/59486535>
5. 解决方案三：
在仓库中找到相关的jar包添加到/WEB-INF/lib中(最直接最简单)

---