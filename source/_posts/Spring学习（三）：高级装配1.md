---
title: Spring核心框架学习（三）：条件装配,自动装配歧义处理与Bean的作用域
date: 2018-04-21 10:15:52
tags:
- Spring
- Spring核心框架
categories: Java框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)

2. 内容概述：
	- Spring profile
	- 条件化的Bean声明
	- 自动化装配与歧义性
	- bean的作用域

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

**2)三种不同的DataSource类：**
1. 获取嵌入式数据库的DataSource。
JavaConfig配置类代码如下：
		package chap3;
		import javax.sql.DataSource;
		import org.springframework.context.annotation.Bean;
		import org.springframework.context.annotation.Configuration;
		import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
		@Configuration
		public class SqlConfig {
			@Bean
			public DataSource dataSource(){
				return new EmbeddedDatabaseBuilder()
				.addScript("classpath:schema.sql")
				.addScript("classpath:test-data.sql")
				.build();
			}
		}
Hypersonic数据库创建的数据库的模式定义在schema.sql中，测试数据则是通过test-data.sql加载的。
在开发环境中运行集成测试或者启动应用进行手动测试的时候，这样创建DataSource是非常有用的。每次启动它的时候都能让数据库处于一个给定(开启服务)的状态。
2. 创建JNDI管理的DataSource。
尽管只使用EmbeddedDatabaseBuilder创建的DataSource非常实用与开发环境，但是对于环境却并不是那么好用。所以一般使用JNDI容器获取的DataSource，但是对于简单的开发测试环境JNDI会带来不必要的复杂性：
		@Bean
		public DataSource dataSource(){
			//使用jndi从容器获取
			JndiObjectFactoryBean jndiObjectFactoryBean=new JndiObjectFactoryBean();
			jndiObjectFactoryBean.setJndiName("jdbc/kenshine");
			jndiObjectFactoryBean.setResourceRef(true);
			jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
			return (DataSource) jndiObjectFactoryBean.getObject();
通过JNDI来获取DataSource能让容器(context)决定通过什么方式获取DataSource。(详细的JNDI配置数据源参考另一篇博客)
这里的JndiObjectFactoryBean仅仅只是通过JNDI名来产生一个DataSource的对象而已。
3. 创建自配置DBCP连接池的DataSource。
在QA环境中可以将DataSource配置为Commons DBCP连接池：
		@Bean
		public DataSource dataSource(){
			BasicDataSource dataSource=new BasicDataSource();
			dataSource.setUrl("jdbc:h2:tcp://dbserver/~/test");
			dataSource.setDriverClassName("org.h2.Driver");
			dataSource.setUsername("sa");
			dataSource.setPassword("password");
			dataSource.setInitialSize(20);
			dataSource.setMaxActive(30);
			return dataSource;
		}
4. 这三个方法分别使用了三种不同的方式来创建DataSource对象。
在不同的环境下所要求的Bean是不同的，所以必须要有一种方式来配置DataSource,来为每一种环境选择合适的配置。
5. 可以在单独的配置类或者xml文件中配置每个bean,然后在**构建阶段**确定要将哪个配置编译到可部署的应用中。但是有一个很大的缺点，就是要为每种环境重新构建应用。(一般是通过Maven Profile确定的)
6. Spring提供的解决方案不需要重新构建--Spring profile。

---
## 2.Profile Bean的配置
其实和使用maven的profile确定环境差不多，只是不是在构建时确定，而是在运行时确定要加载哪一个bean。

**1)JavaConf配置profileBean：**
1. 使用@profile注解：
		@Configuration
		public class SqlConfig {
			@Bean(destroyMethod="shutdown")
			@Profile("dev")
			public DataSource dataSource(){
				return new EmbeddedDatabaseBuilder()
				.addScript("classpath:schema.sql")
				.addScript("classpath:test-data.sql")
				.build();
			}
			@Bean
			@Profile("pro")
			public DataSource dataSource2(){
				//使用jndi从容器获取
				JndiObjectFactoryBean jndiObjectFactoryBean=new JndiObjectFactoryBean();
				jndiObjectFactoryBean.setJndiName("jdbc/kenshine");
				jndiObjectFactoryBean.setResourceRef(true);
				jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
				return (DataSource) jndiObjectFactoryBean.getObject();
			}
			@Bean
			@Profile("qa")
			public DataSource dataSource3(){
				BasicDataSource dataSource=new BasicDataSource();
				dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/springtest?useUnicode=true&characterEncoding=utf-8");
				dataSource.setDriverClassName("com.mysql.jdbc.Driver");
				dataSource.setUsername("root");
				dataSource.setPassword("zjx1754294529");
				dataSource.setInitialSize(20);
				return dataSource;
			}
		}
2. `@Bean(destoryMethod="shutdown")`的含义：
在该bean销毁前去执行shutdown方法。
相应的initMethod就是初始化Bean之前执行的方法。
3. @Profile()注解：
指定某个bean属于哪一个profile,只有当对应的环境激活(active)时才会加载这个bean。
而profile默认是关闭的(未激活的)。
4. 可以直接在方法上使用@profile注解，只有Spring3.2以后支持，Spring3.1及以前需要使用类级别的@profile注解，为每个方法单独配置一个配置类再引用到主配置类。如：
		@Configuration
		@Profile("dev")
		public class EmbeddedConfig {
			@Bean(destroyMethod="shutdown")
			private DataSource dataSource(){
				return new EmbeddedDatabaseBuilder()
				.addScript("classpath:schema.sql")
				.addScript("classpath:test-data.sql")
				.build();
			}
		}
其余两个环境也相同配置。不太方便。
5. 没有指定@profile的Bean始终都会被创建。

**2)XML配置profileBean：**
1. xml配置如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans
		    xmlns="http://www.springframework.org/schema/beans"  
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xmlns:p="http://www.springframework.org/schema/p" 
		    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
		    xmlns:jee="http://www.springframework.org/schema/jee"
		    xmlns:c="http://www.springframework.org/schema/c"  
		    xmlns:context="http://www.springframework.org/schema/context"  
		    xmlns:aop="http://www.springframework.org/schema/aop"   
		    xsi:schemaLocation="http://www.springframework.org/schema/beans   
		                        http://www.springframework.org/schema/beans/spring-beans.xsd  
		                        http://www.springframework.org/schema/aop    
		                        http://www.springframework.org/schema/aop/spring-aop.xsd  
		                        http://www.springframework.org/schema/jdbc    
		                        http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		                        http://www.springframework.org/schema/jee    
		                        http://www.springframework.org/schema/jee/spring-jee.xsd  
		                        http://www.springframework.org/schema/context       
		                        http://www.springframework.org/schema/context/spring-context.xsd">
		   <beans profile="dev">
		   		<jdbc:embedded-database id="dataSource">
		   			<jdbc:script location="classpath:schema.sql"/>
		   			<jdbc:script location="classpath:test-data.sql"/>
		   		</jdbc:embedded-database>
		   </beans>
		   <beans profile="prod">
		   		<jee:jndi-lookup id="dataSource" 
		   		jndi-name="jdbc/kenshine" 
		   		resource-ref="true" 
		   		proxy-interface="javax.sql.DataSource">
		   		</jee:jndi-lookup>
		   </beans>
		   <beans profile="qa">
		   		<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close"
		   		p:url="jdbc:mysql://127.0.0.1:3306/springtest"
		   		p:driverClassName="com.mysql.jdbc.Driver"
		   		p:username="root"
		   		p:password="zjx1754294529"
		   		p:initialSize="20"
		   		>	
		   		</bean>
		   </beans>
		</beans>
2. 使用嵌套的Beans标签分别配置了三种不同profile下的bean。
这三个Bean的id都是相同的，在创建时会根据激活的profile来创建。
也可以不使用嵌套Beans标签，直接在xsi:schemaLocation属性后添加profile属性并配置，但是这样会产生非常多不必要的代码。

**3)关于两种配置方式的总结：**
1. javaConfig配置profile:
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-01-1.jpg)
2. XML配置profile：
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-02.jpg)

---
## 3.激活Profile及使用
**1)Spring激活Profile的原理：**
1. Spring确定哪个profile处于激活状态，需要两个独立的属性：
spring.profiles.active属性和spring.profiles.default属性。
2. 如果active属性指定了激活的profile则激活active指定的profile，
如果active属性未指定激活的profile则激活default指定的profile，
都未指定则只加载那些没有指定profile的Bean。
3. spring.profiles.active属性和spring.profiles.default属性都是复数的，意味着可以一次性激活多个(彼此不相关)的profile。

**2)激活profile的方式：**
有很多种方式能够激活profile(设置那俩属性)。
>作为Spring MVC DispactcherServlet的初始化参数
作为Web应用上下文参数
作为JNDI条目
作为环境变量
作为JVM的系统属性
在集成测试类上，使用@ActiveProfiles注解设置

1. Servlet2.5及以下在web.xml中配置：
		<?xml version="1.0" encoding="UTF-8"?>
		<web -app version="2.5"
		...>
		 <!--为上下文设置默认的profile-->
		 <context-param>
		 <param-name>spring.profile.default</param-name>
		 <param-value>dev</param-value>
		 </context-param>
		 
		...
		 <servlet>
		 ...
		 <!--为Serlvet设置默认的profile-->
		 <init-param>
		  <param-name>spring-profiles.default</param-name>
		  <param-value>dev</param-value>
		 </init-prama>
		...
		<web-app>
这样配置后默认的都是使用的dev开发环境，而实际使用时只需要设置相对应的spring.profile.active的代码就可以了。
2. Servlet3.0以上可以使用如下方式激活：
		package chap3;
		import javax.servlet.ServletContext;
		import javax.servlet.ServletException;
		import org.springframework.web.WebApplicationInitializer;
		public class WebInit implements WebApplicationInitializer{
			@Override
			public void onStartup(ServletContext container) throws ServletException {
				container.setInitParameter("spring.profile.default","dev");
			}
		}
3. JVM增加参数spring.profiles.active设置
在JVM中增加参数spring.profiles.active设置，如果我们想设置spring.profiles.active为dev，使用Dspring.profiles.active="dev" 。
此种方式需要修改tomcat的JVM配置，通用性不高。
4. 在测试时激活：`@ActiveProfiles()`注解
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(locations="classpath:DataSourceConfig.xml")
		@ActiveProfiles("pro")
		public class TestActiveProfile {
			@Autowired
			private HelloService hs;
			
			@Test
			public void testProfile() throws Exception {
				String value = hs.sayHello();
				System.out.println(value);
			}
		}

---
## 4.@Conditional条件化的Bean
**1)简单使用：**
1. @Conditional的配置更加灵活，可以实现在某一个Bean创建之后才创建这个Bean这种效果。
2. Java配置类的写法：
		@Configuration
		public class ConditionConfig {
			@Bean
			@Conditional(GoodMusicCondition.class)  //一个实现了Conditon接口的类
			public GoodMusic goodMusic(){
				return new GoodMusic();
			}
		}
3. GoodMusicCondition类的写法：
		public class GoodMusicCondition implements Condition{
			@Override
			public boolean matches(ConditionContext context,
					AnnotatedTypeMetadata metadata) {
				//各项检测机制,返回true或false代表是否生成Bean
				Environment env=context.getEnvironment();
				return env.containsProperty("softMusic");
			}
		}
检测上下文中是否已经创建了名为softMusic的Bean，如果有则装配GoodMusic的Bean，没有则不装配。
还可以在这里面写更复杂的逻辑。

**2)关于Condition接口的介绍：**
1. Condition接口：
		public interface Condition {
			boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
		}
可以看到matches方法是由ConditionContext对象与AnnotatedTypeMetadata对象来操作的，这俩其实都是接口
2. ConditionContext接口：
		public interface ConditionContext {
			BeanDefinitionRegistry getRegistry();	//检查bean的定义
			ConfigurableListableBeanFactory getBeanFactory();	//检查bean是否存在并探查bean的属性
			Environment getEnvironment();	//环境变量是否存在以及值是什么
			ResourceLoader getResourceLoader();		//返回ResourceLoader加载的资源
			ClassLoader getClassLoader();	//返回ClassLoader加载并检查类是否存在
		}
3. AnnotatedTypeMetadata接口：
可以检查带有@Bean的注解方法上还有什么其他注解。
		import org.springframework.util.MultiValueMap;
		
		public interface AnnotatedTypeMetadata {
		
			boolean isAnnotated(String annotationName);
		
			Map<String, Object> getAnnotationAttributes(String annotationName);
		
			Map<String, Object> getAnnotationAttributes(String annotationName, boolean classValuesAsString);
		
			MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationName);
		
			MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationName, boolean classValuesAsString);
		
		}

**3)@Profile的实现：**
1. profile的代码：
		@Target({ElementType.TYPE, ElementType.METHOD})
		@Retention(RetentionPolicy.RUNTIME)
		@Documented
		@Conditional(ProfileCondition.class)
		public @interface Profile {
			String[] value();
		}
2. 其余注解以后研究源码再说，先看ProfileCondition类：
		class ProfileCondition implements Condition {
			@Override
			public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
				if (context.getEnvironment() != null) {
					MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
					if (attrs != null) {
						for (Object value : attrs.get("value")) {
							if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
								return true;
							}
						}
						return false;
					}
				}
				return true;
			}
		}
使用getAllAnnotationAttributes获取@profile注解的所有属性，检查value属性并获取profile名称，最后通过acceptsProfiles方法来判断该profile是否激活。

---
## 5.处理自动装配时的歧义性
**1)歧义性产生情况及解决方法概述：**
1. 歧义产生的简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-03-2.jpg)
2. 产生歧义的代码示例：
Movie代码：
		@Component
		public class Movie1 implements Movie{
			@Override
			public void play() {
				System.out.println("playing movie1");
			}
		}
		@Component
		public class Movie1 implements Movie{
			//和上面差不多
		}
		@Component
		public class Movie1 implements Movie{
			//和上面差不多
		}
MoviePlayer类代码：
		@Component
		public class MoviePlayer implements MediaPlayer{
			@Autowired
			private Movie movie;
			@Override
			public void play() {
				movie.play();
			}
		}
MoviePlayerConfig配置类：
		@Configuration
		@ComponentScan(basePackageClasses=MoviePlayer.class)
		public class MoviePlayerConfig {
		}
3. 测试类：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MoviePlayerConfig.class)
		public class MoviePlayerTest {
			@Autowired
			private MoviePlayer moviePlayer;
			@Test
			public void play(){
				moviePlayer.play();
			}
		}
测试结果：
报错：
		No qualifying bean of type 'chap3.Movie' available: expected single matching bean but found 3: movie1,movie2,movie3
4. 有两种解决方案：
标识首选的bean。
使用限定符缩小范围。

**2)使用primary注解或者属性标识首选的bean：**
1. 配置方式一：
在希望成为首选Bean的@Compnent下使用@primary注解:
		@Component
		@Primary
		public class Movie1 implements Movie{
			@Override
			public void play() {
				// TODO Auto-generated method stub
				System.out.println("playing movie1");
			}
		}
2. 配置方式二：
显式配置javaConfig配置类并且在@Bean注解下使用@primary注解指定：
		@Configuration
		public class MovieConfig {
			@Bean
			@Primary
			public Movie1 movie1(){
				return new Movie1();
			}
			@Bean
			public Movie2 movie2(){
				return new Movie2();
			}
			@Bean
			public Movie3 movie3(){
				return new Movie3();
			}
		}
3. 配置方式三：显式的XML配置
		...省略配置头
		<bean id="movie1" class="chap3.Movie1" primary="true"></bean>
		<bean id="movie2" class="chap3.Movie2"></bean>
		<bean id="movie3" class="chap3.Movie3"></bean>
		...省略配置尾
4. 以上任意一种配置的测试结果都是这样：
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-04.jpg)
4. 但是如果有两个以上的primary还是会错，所以推荐使用限定符修饰。

**3)限定自动装配的Bean：**
1. 使用自带的注解：
在MoviePlayer中使用@Qualifier注解：
		@Autowired
		@Qualifier("movie1")
		public Movie movie;
就是按照上下文中的ID来获取,ID默认是@Bean方法名的第一个字母小写，所以这里使用movie1。
但是这样有个缺陷：
如果方法名字修改后如将Movie1修改为其他的，那么久无法获取到bean了。
2. 创建自定义的限定符(id)
可以在Movie1中这样修改(使用自定义的ID)：
		@Component
		@Qualifier("movie1")
		public class Movie1 implements Movie{
			@Override
			public void play() {
				// TODO Auto-generated method stub
				System.out.println("playing movie1");
			}
		}
也可以在显示声明Bean时这样使用：
		@Bean
		@Qualifier("movie1")
		public Movie1 movie1(){
			return new Movie1();
		}
或者：
		@Bean(name="movie1")
		public Movie1 movie1(){
			return new Movie1();
		}
但是这种方式有时候还是有问题，比如不知道是否有重复的ID。
这是就得使用多个注解修饰了。

**4)自定义限定注解的使用：**
1. 如果说有两个bean的@Qualifier修饰相同，那么取的时候仍然会报错，一般会想到再加一个@Qualifier注解进行修饰，然后取的时候也再加一个@Qualifier注解如：
		@Autowired
		@Qualifier("movie1")
		@Qualifier("comedy")
		public Movie movie;
但是Java8规定只有使用了@Repeatable注解的注解才能重复，而很遗憾@Qualifier注解并没有：
		@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Inherited
		@Documented
		public @interface Qualifier {
			String value() default "";
		}
所以使用两个@Qualifier会出错。所以需要自定义一种注解（比如@Comedy）。
2. 自定义注解@comedy的创建：
代码如下：
		@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Qualifier
		public @interface Comedy {
		}
3. 修改Movie2代码修改如下：
		@Component
		@Qualifier("movie2")
		@Comedy
		public class Movie2 implements Movie {
			@Override
			public void play() {
				// TODO Auto-generated method stub
				System.out.println("playing movie2");
			}
		}
修改MoviePlayer代码如下：
		@Component
		public class MoviePlayer implements MediaPlayer{
			@Autowired
			@Qualifier("movie2")
			@Comedy
			public Movie movie;
			@Override
			public void play() {
				movie.play();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-05.jpg)
4. 如果将某一个@Comedy标签去除，不管去掉啥标签，只要是不完全匹配就会报错说找不到对应的Bean。

---
## 6.Bean的作用域
**1)作用域Bean简介及简单创建：**
1. 在默认情况下，Spring上下文中的所有Bean都是以单例的情况存在的（单例bean）,不管获取多少次上下文中的Bean,获取到的Bean都是同一个实例。
大多数情况下这种单例的方案很好，但是有时有特殊需求时就不那么好用了。
2. Spring自定义了很多种作用域，可以基于这些作用域创建Bean:
	- 单例（Singleton）：整个应用中只能创建一个Bean实例。
	- 原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候都会创建一个新的Bean。
	- 会话(Session)：WEB应用中，为每个会话创建一个bean实例。
	- 请求(Request)：WEB应用中，为每个请求创建一个bean实例。
3. 使用@Scope注解或者属性可以指定作用域生成相对的Bean。
方法一：自动装配置时生成原型Bean
		@Component
		@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)	//也可以直接使用("prototype")
		public class Movie4 implements Movie{
			@Override
			public void play() {
				System.out.println("playing movie4");
			}
		}
方法二：JavaConfig配置类配置@Scope
		...
		@Bean
		@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
		public Movie4 movie4(){
			return new Movie4();
		}
		...
方法三：config.xml文件中使用scope属性配置
		...
		<bean id="movie4" class="chap3.Movie4" scope="prototype"></bean>
		...

**2)使用会话和请求作用域：**
1. 如果是一个购物车的Bean,单例Bean则所有用户都使用一个Bean,而原型Bean则在应用的其他地方就不可用了。用SessionBean就非常合适。
在WEB应用中，会话级和请求级的Bean非常实用。
2. 购物车类Cart如下：
		@Component
		@Scope(value=org.springframework.web.context.WebApplicationContext.SCOPE_SESSION,proxyMode=ScopedProxyMode.INTERFACES)
		public class Cart implements ShoppingCart{
			public String name="Cart1";
			@Override
			public void getName() {
				System.out.println(name);
			}
		}
接口如下：
		public interface ShoppingCart {
			void getName();
		}
@Scope注解还有个方法proxyMode，解决了将会话或请求作用域的bean注入到单例bean中所遇到的问题。
3. 创建一个单例Bean类StoreService：
		@Component
		public class StoreService {
			@Autowired
			public ShoppingCart shoppingCart;
			
			public void Service(){
				shoppingCart.getName();
			}
		}
测试类如下：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ComponentScan
		public class CartTest {
			@Autowired
			private StoreService storeService;
			@Test
			public void testCart(){
				storeService.Service();
			}
		}
4. StoreService是一个单例类，会在Spring应用上下文加载的时候创建。
创建时会试图将ShoppingCartBean注入单例类中，但是会话级的bean此时不存在，只有用户进入了才会产生实例。
5. 多个用户会有多个ShoppingCart，但是我们只希望注入当前会话对应的哪一个。
Spring并不会将实际的ShoppingCart注入StoreService，而是注入一个ShoppingCart Bean的代理。
6. 关于proxyMode的值：
	- 如果注入的是接口而不是类(最好的状态)，则使用ScopedProxyMode.INTERFACES。
	表明该代理要实现这个接口并调用委托给实现类。
	- 如果注入的是具体类而非接口，则proxyMode值使用ScopedProxyMode.TARGET_CLASS。使用的是CGLib动态代理。
7. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00044Spring%E5%AD%A6%E4%B9%A03-06.jpg)

**3)在XML中声明作用域代理：**
1. 使用aop命名空间的scoped-proxy元素(需要)：
		 <bean id="cart" class="chap3.ShoppingCart" scope="session">
		 	<aop:scoped-proxy/>
		 </bean>
2. 默认是基于具体类的代理，如果需要生成基于接口的代理可以这么写：
		 <bean id="cart" class="chap3.ShoppingCart" scope="session">
		 	<aop:scoped-proxy proxy-target-class="false"/>
		 </bean>

---