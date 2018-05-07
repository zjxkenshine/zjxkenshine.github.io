---
title: Spring核心框架学习（四）：运行时值注入以及SpEL
date: 2018-04-22 22:10:39 
tags:
- Spring
- Spring核心框架
- SpEL
categories: Java框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)

2. 内容概述：
	- 运行时值注入的两种方式
	- 属性占位符
	- Spring表达式语言：
	Spring Expression Language(SpEL)

---
## 1.运行时值注入简介
1. 讨论依赖注入时更多的是只将一个Bean注入到另一个Bean的构造方法或者Setter方法中，通常指的是一个对象与另一个对象相关联。
2. 依赖注入的另一个含义是将值注入到Bean的属性或者构造函数的参数中。
3. 可以使用的方式(但是太不灵活，只是固定的)：
		@Bean
		public Music newmusic(){
			return new NewMusic("happy","kenshine");
		}
xml：
		<bean id="happyMusic" class="chap2.HappyMusic" c:song="happy" c:singer="kenshine">
		</bean>
4. 如果是一个变量值需要注入到Bean中该怎么做呢?
有两种方式：
	- 属性占位符
	- Spring表达式语句(SpEL)

---
## 2.使用Environment注入外部值
**1)使用Environment从外部属性源读取：**
处理外部值最简单的方式就是声明属性源并用Environment检索属性。
1. 创建Movie的实现类GoodMovie：
		public class GoodMovie implements Movie {
			private String name;
			public GoodMovie(String name) {
				this.name = name;
			}
			@Override
			public void play() {
				System.out.println("Playing goodMovie "+name);
			}
		}
2. 在src/main/resources创建mov.properties，内容如下:
		mov_name = 007
3. 创建GoodMusicConfig.java内容如下：
		@Configuration
		@PropertySource("classpath:mov.properties")	//声明属性注入源
		public class GoodMovieConfig {
			@Autowired
			Environment env;
			@Bean
			@Qualifier("goodMovie")
			public Movie GoodMovie(){
				return new GoodMovie(env.getProperty("mov_name"));  //evn检索并获取属性值
			}
		}
通过getProperty()方法来获取属性文件中的属性。
4. 修改MoviePlayer如下：
		@Component
		public class MoviePlayer implements MediaPlayer{
			@Autowired
			@Qualifier("goodMovie")
			public Movie movie;
			@Override
			public void play() {
				movie.play();
			}
		}
MoviePlayerConfig配置类代码：
		@Configuration
		@ComponentScan(basePackageClasses=MoviePlayer.class)
		public class MoviePlayerConfig {
		}
5. 测试代码：
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
![](http://p5ki4lhmo.bkt.clouddn.com/00046Spring%E5%AD%A6%E4%B9%A04-01.jpg)

**2)深入理解Environment：**
1. 打开Environment时会发现它是一个接口，并有以下关联的类，接口：
![](http://p5ki4lhmo.bkt.clouddn.com/00046Spring%E5%AD%A6%E4%B9%A04-02.jpg)
2. 上面使用的方法类中Environment的父接口PropertyResolver，代码如下：
		public interface PropertyResolver {
			//属性是否存在
			boolean containsProperty(String key);
			//获取属性(未定义不报错)
			String getProperty(String key);
		
			String getProperty(String key, String defaultValue);
		
			<T> T getProperty(String key, Class<T> targetType);
		
			<T> T getProperty(String key, Class<T> targetType, T defaultValue);
			
			@Deprecated
			<T> Class<T> getPropertyAsClass(String key, Class<T> targetType);
			//获取属性(未定义报错)
			String getRequiredProperty(String key) throws IllegalStateException;
			
			<T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;
			
			String resolvePlaceholders(String text);
			
			String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;
		}
可以发现有getProperty方法有四个重载形式，所以可以使用。
3. 其余三个重载getProperties()方法的介绍
`String getProperty(String key, String defaultValue)`：指定属性不存在时会使用后面指定的默认值,如：
		env.getProperty("mov_name"，"TaiTanic")；
使用这两个方法获得的值都是String类型的，而后面两个重载方法返回的值时泛型，不再局限于String类型，有时比较方便:
		<T> T getProperty(String key, Class<T> targetType);
		<T> T getProperty(String key, Class<T> targetType, T defaultValue);
4. 其余方法介绍：
	- 如果在该例子中mov_name属性不存在(未定义)，不会报错，会返回null;
	`getRequiredProperty(...)`方法：要求属性一定存在，不存在则报错。
	- `containsProperty(String key);`：属性是否存在或已定义。
	- `getPropertyAsClass(...)`:将属性解析为类并返回
5. 接口自带的方法：
		public interface Environment extends PropertyResolver {
			//返回激活的Profiles名称的数组
			String[] getActiveProfiles();
			//返回默认的profile名称的数组
			String[] getDefaultProfiles();
			//给定的Enviroment支持profiles,则返回true
			boolean acceptsProfiles(String... profiles);
		}
6. 关于Environment的总结：
使用Environment的方法还是很有好处的。
在Java配置中装配Bean的时候直接从Environment中检索属性是非常方便的。
7. 简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00046Spring%E5%AD%A6%E4%B9%A04-03-1.jpg)

---
## 3.属性占位符实现值注入
**1)使用属性占位符：xml方式**
1. Spring一直支持将属性定义到外部的属性文件中，并使用占位符将值插入到SpringBean中。
在Spring装配中使用${..}的方式包装属性。
2. 在XML配置文件中可以这么写：
		<context:property-placeholder location="classpath*:mov.properties" />
		<bean id="goodMovie" class="chap3.GoodMovie" c:name="${mov_name}"></bean>
`<context:property-placeholder>`标签生成了一个ProPertySourcePlaceholderConfigurer Bean。
该类能够基于Spring Enviroment及其属性源来解析占位符。
location属性指定属性源。
3. `<context:property-placeholder location="classpath*:mov.properties" />`是配置一个数据源，也可以使用以下方式：
		<bean id="propertyConfigurer"class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
		   <property name="location">
		     <value>mov.properties</value>
		   </property>
		    <property name="fileEncoding">
		      <value>UTF-8</value>
		    </property>
		</bean>
4. 配置多个数据源可以这么写：
		<bean id="propertyConfigurer"class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">  
		    <property name="locations">
		        <list>
		            <value>/WEB-INF/mail.properties</value>  
		            <value>classpath: conf/sqlmap/jdbc.properties</value>//注意这两种value值的写法
		     	</list>
		    </property>
		</bean>
5. Spring3.2之后推荐使用PropertySourcesPlaceholderConfigurer。

**2)使用属性占位符：Java方式**
1. 使用@Value标签，用法与@Autowired类似：
在GoodMovie中使用。
用法一：
		@Component
		@Qualifier("goodMovie")
		public class GoodMovie implements Movie {
			private String name;
			public GoodMovie(@Value("${mov_name}") String name) {
				this.name = name;
			}
			@Override
			public void play() {
				System.out.println("Playing goodMovie "+name);
			}
		}
用法二：
		@Value("${mov_name}")
		private String name;
还可以用在Setter方法上。
2. 将MoviePlayerConfig.java修改如下：
		@Configuration
		@ComponentScan(basePackageClasses=MoviePlayer.class)
		@PropertySource("classpath:mov2.properties")	//声明属性注入源
		public class MoviePlayerConfig {
			@Bean
			public PropertySourcesPlaceholderConfigurer configurer(){
				return new PropertySourcesPlaceholderConfigurer();
			}
		}
mov2.properties的mov_name是008。
也可以在GoodMovieConfig.Java中配置。
3. 测试类和上面的相同。
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00046Spring%E5%AD%A6%E4%B9%A04-04.jpg)
4. 为了使用占位符最好加上加上PropertySourcesPlaceholderConfigurer的Bean配置。(或PropertylaceholderConfigurer)
注意spring3.2以后推荐使用PropertySourcesPlaceholderConfigurer。
(3.1及以前使用PropertylaceholderConfigurer)
5. 简单的理解：
只要使用`${属性}`然后在任意一个地方声明数据源到Enviroment就可以了。
最好加上PropertySourcesPlaceholderConfigurer的Bean配置。
可能使用了@PropertySource注解就不需要PropertySourcesPlaceholderConfigurer的Bean配置了。

**3)PropertySourcesPlaceholderConfigurer工作原理：**
暂时看不懂，可以参考大佬的博客：
[Spring PropertySourcesPlaceholderConfigurer工作原理](https://blog.csdn.net/xczzmn/article/details/77744627)

---
## 4.SpEL表达式的简单使用
1. Spring3引入了Spring的表达式语言，能够以一种强大和简洁的方式将值装配到Bean属性和构造器参数中，在这个过程中使用的表达式会在计算是计算的到值。
2. SpEL拥有很多特性：
	- 使用bean的ID来引用bean;
	- 调用方法和访问对象的属性；
	- 对值进行算术，关系和逻辑运算；
	- 正则表达式匹配；
	- 操作集合。
3. 除了这些基本用法外SpEL还可以用在依赖注入的其他地方。如Spring Scurity支持使用SpEL定义安全限制规则，在Spring MVC中可以引用Thymeleaf数据。

### 使用示例
**1)SpEL基础知识,基本用法：**
1. 基本用法：`#{..}`
和占位符的`${..}`类似
如最简单的常量值：`#{1}`
2. 比较特别一点的：
`#{T(System).currentTimeMillis}`
T()表示里面是java的类，这句话相当于注入了System.currentTimeMillis的值。
3. 也可以引用其他bean或者属性：
`#{goodMethod.name}`
还可以通过systemProperties对象引用系统属性：
`#{systemProperties['mov_name']}`
4. 这只是一些简单的例子，后面还会涉及更多。

**2)在Bean装配时使用这些表达式：**
1. 在Java类中，需要搭配@Value注解来使用：
		public GoodMovie(@Value("#{systemProperties['mov_name']}") String name) {
			this.name = name;
		}
2. 也可以在XML中配置Bean：
		<bean id="goodMovie" class="chap3.GoodMovie" 
		c:name="#{systemProperties['mov_name']}">
		</bean>
或者使用`<constructor-arg>`标签或者`<properity>`标签的value属性。
3. 以上的配置可以直接从系统属性中获得mov_name的值并注入到bean中。
4. 关于系统属性：
系统属性是以键值对的方式存储的。
可以使用System.getProperties()来查看，真的很多，我就不贴了。
也可以使用如下代码来格式化查看：
		 Properties props=System.getProperties();
		  Iterator iter=props.keySet().iterator();
		  while(iter.hasNext())
		  {
		  String key=(String)iter.next();
		  System.out.println(key+" = "+ props.get(key));
		  }
		 }
5. 设置系统属性：
可以使用：
		System.setProperty("key","value");

### 字面值,引用bean,属性及方法
**1)字面值：**
1. SpEL可以用来表示浮点数，String以及Boolean值。
2. 简单例子：
浮点：
		#{3.1415...}
使用科学计数法：
		#{9.8E4}
计算String类型的字面值：
		#{'Hello'}
将true,false转换为对应的boolean类型：
		#{true}
3. SpEL在使用字面量时没有什么意义，一般是用在复杂的表达式中。

**2)引用Bean，属性和方法：**
1. 通过ID来引用其他Bean：
		#{goodMovie}
还可以使用bean的属性(如goodMovie的name)：
		#{goodMovie.name}
2. 调用bean的方法：
		#{goodMovie.getName()}
甚至可以调用方法返回值类的方法：
		#{goodMovie.getName().trim().toUpperCase()...}
3. 如果上面的方法返回值时null就会出错，所以可以使用类型安全的运算符`?.`：
如：
		#{goodMovie.getName()?.trim()?.toUpperCase()...}
这样如果返回值时null就不会执行后面的代码而是直接放回null。

### 使用类型及运算符
**1)在表达式中使用类型：**
1. 使用`T()`方法，如：
		T(java.lang.Math)
会返回一个class对象，可以将其注入到一个Bean中。
但是最主要的功能是可以使用该类中的静态方法以和常量。
2. 如将PI的值装配到Bean中：
		#{T(java.lang.Math).PI}
随机数：
		#{T(java.util.Random).random}

**2）SpEL运算符：**
1. 简单图示：
![](http://p5ki4lhmo.bkt.clouddn.com/00046Spring%E5%AD%A6%E4%B9%A04-05.jpg)
2. 求圆周长：2πr
		#{2*T(java.lang.Math).PI*circle.radius}
求圆面积：πr^2
		#{T(java.lang.Math).PI*circle.radius^2}
3. 连接字符串：+
		#{'name='+name}
判断是否相等：
`#{a==b}`或者`#{a eq b}`
4. 三元运算符：
		#{a>b?1:0}
三元运算符经常会用于检测null，并使用默认值代替null：
		#{name?:default}

### 处理正则表达式与集合
**1）计算正则表达式：**
1. 需要使用matches运算符：`String matches 正则表达式`
2. 如：
		#{name match [a-z]}
3. 更多的正则表达式的语法有空在整理。

**2)操作数组与集合：**
1. 引用数组中的一个元素：
		#{songs[4].play()}
还可以加一些复杂的运算：
		#{songs[T(java.lang.Math).random()*songs.length()].singer}
2. `[]`运算符从集合或者数组按照索引获取元素。还可以获取字符串的字符：
		#{'hello world'[4]} //0
3. SpEL甚至还能对集合进行过滤，通过查询运算符`.?[]`：
		#{songs.?[singer=='kenshine']}
上面的查询运算符是查询全部的。SpEL还有查询首个和查询最后一个的查询：
`.^[]`(首)和`.$[]`(末)
如查询第一首歌手是kenshine的歌：
		#{songs.^[singer='kenshine']}
4. SpEL甚至还提供了一个我听都没听说过的投影运算符：`.![]`
它会从集合中的每个成员中选择特定的属性放到另一个集合中。
如：获取kenshine的所有歌曲列表
		#{songs.?[singer==kenshine].![title]}
而对上述得出的集合又可以进行上面的所有操作。

**3)SpEL总结：**
这些只是SpEL的皮毛，更多的SpEL操作以后再深入学习了。

---

