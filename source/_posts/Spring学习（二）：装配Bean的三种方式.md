---
title: Spring核心框架学习（二）：装配Bean的三种方式
date: 2018-04-16 19:22:11
tags:
- Spring
- Spring核心框架
categories: J2EE框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)

2. 内容概述：
	- 声明Bean
	- 构造器注入和Setter注入
	- 装配Bean
	- 控制Bean的创建和销毁

---
## 1.装配Bean的三种方式简介
**1)装配简介：**
1. 任何一个成功的应用都是由多个为了实现某一个业务二相互协作的组件构成。这些组件必须彼此了解，相互协作来完成工作。
2. 创建对象之间的协作关系的行为通常称为装配(wiring),这也是依赖注入的本质。
3. 在开发基于Spring的应用开发时随时都会使用这些技术。

**2)装配Bean的三种方式简介及选择：**
1. 三种方式简介：
	- 在XML中进行显式配置
	- 在Java中进行显式配置
	- 隐式的bean发现机制和自动装配
2. 如何选择：
选择哪种方案很多时候是个人喜好，没有太大区别，不过为了简化配置尽量使用自动装配。
3. 三种装配范式是可以互相搭配的，所以可以选择使用xml装配一些bean,使用Spring基于Java的配置装饰另一些bean,剩余的bean使用自动装配。

---
## 2.自动装配Bean
**1)总体介绍：**
1. Spring从两个大角度来实现自动化装配：
	- **组件扫描**：Spring自动发现应用上下文中所创建的bean
	- **自动装配**：Spring自动满足bean之间的依赖
2. 组件扫描和自动装配组合在一起就能将显示的配置降低到最低。
3. 我的大致理解：(详细见后面)
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-01.jpg)

### 实现自动组件扫描
**1)组件扫描：**
假设是一个音乐类的Bean。
1. 创建音乐接口(Music)：
		package chap2;
		public interface Music {
			void play();
		}
创建实现类SoftMusic实现Music，并声明为一个组件：
		package chap2;
		import org.springframework.stereotype.Component;
		
		@Component
		public class SoftMusic implements Music {
			private String song="sad";
			private String singer="kenshine";
			@Override
			public void play() {
				System.out.println("playing song "+song+",singer : "+singer);
			}
		}
需要注意的是只要在创建实现类时加上@Component注解声明为一个组件，Spring就会自动为该实现类创建Bean(通过组件扫描),而不用显式的配置。
2. 启用组件扫描的两种方式：
组件扫描默认是不启用的，所以只配置了@Component是没有效果的。
启用组件扫描有两种方式：java配置方式与xml配置方式。
3. java配置类方式开启扫描组件：
创建音乐播放器(MusicPlayer)的配置文件，MusicPlayer类会在后面创建，会将Music的实现类注入进行播放,在该配置文件中启用组件扫描：
		package chap2;
		
		@Configuration
		@ComponentScan
		public class MusicPlayerConfig {
		}
这里的MusicPlayerConfig没有显示声明任何bean,只是使用了@ComponentScan启用组件扫描。
如果没有其他配置的话，@ComponentScan会默认扫描配置类所在的包与其所有的子包，会扫描其中带有@Component注解的类并为其自动创建一个bean。
4. xml方式启用组件扫描：
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
			
			<context:component-scan base-package="chap2"></context:component-scan>
		
		</beans>
默认扫描的是base-package属性配置的包。
后面的配置都是以java配置类的方式，不过`<context:component-scan>`标签有与@ComponentScan注解相对应的属性和标签。
5. 简单测试:
测试组件扫描是否能发现@Component并创建对应的bean：
		package chap2;
		import org.junit.Assert;
		import org.junit.Test;
		import org.junit.runner.RunWith;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.test.context.ContextConfiguration;
		import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MusicPlayerConfig.class)
		public class ComponentScanConfigTest {
			
			@Autowired
			private SoftMusic softMusic;
			
			@Test
			public void softMusicIsNotNull(){
				Assert.assertNotNull(softMusic);
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-02.jpg)

**2)组件扫描的设置**
1. 为组件扫描的Bean命名：
Sring应用上下文中的bean都会给定一个ID，默认为将类名的第一个字母变为小写，这里就是softMusic。不过这个Id只是为了区分不同的实现类而已，如果把上述测试的对象名softMusic改为别的，测试依然会通过。
设置方式一@Component命名：
		...
		@Component("sad")
		public class SoftMusic implements Music {
			...
		}
测试以上代码依然通过。
设置方式二@Named命名(有前提，见下方)：
		...
		@Named("sad")
		public class SoftMusic implements Music {
			...
		}
Spring支持将@Named作为@Component的替代方案。两者之间有一些细微差别但是绝大多数情况下可以互相替换。
2. 使用@Named注解需要导java.inject包，添加pom.xml配置如下：
		<dependency>
		  <groupId>javax.inject</groupId>
		  <artifactId>javax.inject</artifactId>
		  <version>1</version>
		</dependency>
将@Component替换为@Named后依旧可以测试成功。
3. 设置组件扫描的基础包(java配置类中修改)：
默认扫描的是java配置类所在的包。
将java配置类修改如下：
		package chap2;
		
		@Configuration
		@ComponentScan("chap2")
		public class MusicPlayerConfig {
		}
则可以扫描包chap2（其实配置类就在这个包中），也可以使用更清晰的配置方式：
		@ComponentScan(basePages="chap2")
basePages属性也可以配置扫描多个包：
		@ComponentScan(basePages={"chap2","chap1"})
4. 设置组件扫描的基础包(安全版)：
使用basePagesClasses属性指定保证所含的类或者接口：
		@ComponentScan(basePagesClasses={MusicPlayer.class,	MoviePlayer.class})
这俩类暂时未创建，但是是可以这样子写的。可以防止重构时出错。不过还有一种更好的方法，新建一个MediaPlayer空接口用于标记，然后这样写：
		@ComponentScan(basePagesClasses={MediaPlayer.class})
接口代码如下：
		package chap2;
		public interface MediaPlayer {
			void play();
		}
这样只要接口的位置不变，就都能扫描到MusicPlayer类。
MusicPlayer的代码稍后创建，因为涉及到自动装配。

### 实现自动装配（注入）
**1)自动装配的实现：**
1. 很多时候我们需要有一种方法能将组件扫描的到的bean和它的依赖类装配在一起：**自动装配**。
2. 创建需要依赖注入的类MusicPlayer:
		package chap2;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Component;
		
		@Component
		public class MusicPlayer implements MediaPlayer {
			private Music music;
			@Autowired
			public MusicPlayer(Music music){
				this.music=music;
			}
			@Override
			public void play() {
				music.play();
			}
		}
@Autowired注解可以用在任何方法甚至是属性上用来获取bean。这里使用该方法获取softMusic进行依赖注入(声明依赖)。
3. 不管是什么方法声明了依赖，Spring都会尝试满足方法参数上所声明的依赖。
如果只有一个Bean满足依赖需求，那么就装入；
如果有多个满足，则会拋异常，出现歧义性(高级装配中会细讲)；
如果没有bean满足需求，那么会拋出空异常，可以通过@Autowired(required=false)配置。
4. 同样也可以使用javax.inject的替代方式：@Inject注解：
		...
		@Inject
		public MusicPlayer(SoftMusic softMusic){
			this.softMusic=softMusic;
		}
		...
但是它并没有为匹配为空的时候提供required属性。
不过在大多数场合@Inject和@Autowired注解是可以互相切换的。

**2)自动装配的测试及一些结论：**
1. 首先如果使用到输出作为断言的判断条件，则需要在pom.xml中添加配置如下：
		<dependency>
		  <groupId>com.github.stefanbirkner</groupId>
		  <artifactId>system-rules</artifactId>
		  <version>1.5.0</version>
		</dependency>
2. 测试代码如下：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MusicPlayerConfig.class)
		public class MusicPlayerTest {
			//SystemRule库的一个Junit规则，可以将输出作为断言条件
			@Rule
			public final StandardOutputStreamLog log=new StandardOutputStreamLog();
			
			@Autowired
			private MediaPlayer player;
			
			@Autowired
			private Music music;
			
			@Test
			public void musicNotBeNull(){
				Assert.assertNotNull(music);
			}
			
			@Test
			public void play(){
				player.play();
				//通过log.getLog()得到上面输出的值并进行断言测试
				Assert.assertEquals("playing song sad,singer : kenshine",log.getLog());
			}
		}
不过需要将SoftMuisic的System.out.println改为System.out.print，不然怎么样都测试不通过，测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-03.jpg)
3. 上面的测试在MusicPlayerConfig中我的配置类扫描源是标记接口：
		@ComponentScan(basePackageClasses=MediaPlayer.class)
改为默认的或者某个包等都可以测试通过。
4. 关于StandardOutputStreamLog，这是System Rules库中的一个Junit规则，使用@Rule标记，可以将基于控制台的输出发送到断言控制。
5. 以上代码的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-04-1.jpg)

---
## 3.通过Java代码实现装配bean
**1）为什么要显示的装配bean而且首选java**
1. 尽管自动装配非常好用而且更加推荐，但是有时候使用的是第三方库，如果想要将第三方库中的组件装配到你的应用中，这时就不能使用自动装配了，因为无法修改它的类去添加@Component注解。
2. 进行显式配置时，JavaConfig是最推荐的方案，因为它更为强大，类型安全而且对重构友好。
3. JavaConfig又与一般的Java代码是不一样的，它不涉及业务逻辑与领域，所以一般会将javaConfig配置代码单独放置到一个包中(不强制)。

**2)配置类的创建及简单声明Bean：**
1. 创建MusicPlayer2Config类如下：
		package chap2;
		import org.springframework.context.annotation.Configuration;
		
		@Configuration
		public class MusicPlayer2Config {
		}
JavaConfig类的创建依赖于@Configuration注解，该注解表名这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。
2. 声明简单的bean(在配置类中)：
		@Bean
		public Music getSoftMusic(){
			return new SoftMusic();
		}
@Bean注解会告诉Spring这个方法会返回一个对象，该对象要注册为Spring上下文中的bean。方法体中则包含了最终产生bean的实例。
3. 默认情况下的Bean的ID与带有@Bean的方法时一样的，不过可以通过@Bean注解的name属性来给它设置一个不同的名字：
		@Bean(name="softMusic")
		public Music getSoftMusic(){
			return new SoftMusic();
		}
返回值使用接口就可以了。只要最终生成的是Music的实例就可以了。
4. 在创建Bean的方法体中可以写很复杂的逻辑，只要返回值的类型正确即可。
5. 开启@Bean注解后会**阻断所有**对该方法的调用，

**3)实现注入与测试**
1. 写法一（注入类为MusicPlayer）：
		@Bean
		public MusicPlayer musicPlayer(){
			return new MusicPlayer(getSoftMusic());
		}
但是并不会调用getSoftMusic()方法，而是会把上下文中的bean注入给MusicPlayer。如果再有一个和这个一样的方法：
		@Bean
		public MusicPlayer musicPlayer2(){
			return new MusicPlayer(getSoftMusic());
		}
那么这两个方法所的到的softMusic的Bean是同一个实例。
Spring中的Bean都是单例的，没必要像这样为第二个musicPlay再创建一个Bean。
2. 写法二：(更好的写法)
		@Bean
		public MusicPlayer musicPlayer(Music music){
			return new MusicPlayer(music);
		}
这是引用其他bean的最好写法，不用将Music声明在同一个配置类中。甚至可以不是JavaConfig配置的(组件扫描或者XML配置的)。
3. 带有@Bean注解的方法会采用任何必要的Java功能来产生Bean实例。
4. 测试代码：
		public class MusicPlayerTest2 {
			//SystemRule库的一个Junit规则，可以将输出作为断言条件
			@Rule
			public final StandardOutputStreamLog log=new StandardOutputStreamLog();
			AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(MusicPlayer2Config.class);
			//使用返回值类型匹配
			private MediaPlayer player=context.getBean(MusicPlayer.class);
			//使用设置的name匹配
			private Music music=(Music)context.getBean("softMusic");
			@Test
			public void close(){
				context.close();
			}
			@Test
			public void musicNotBeNull(){
				Assert.assertNotNull(music);
			}
			@Test
			public void play(){
				player.play();
				//通过log.getLog()得到上面输出的值并进行断言测试
				Assert.assertEquals("playing song sad,singer : kenshine",log.getLog());
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-05.jpg)
注意这里的@RunWith以及@ContextConfiguration注解都没有使用。
可以看到有多种方式可以从上下文中取值。
5. 我的JavaConfig配置思路图：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-06.jpg)

---
## 4.通过XML装配Bean
**1)XML配置简介：**
1. Spring刚出现的时候，XML是主要来描述配置的。
2. 但是，XML的时代已经过去了，现在的Spring有了更强大的自动装配以及基于Java的装配。但是仍需要了解XML配置来对老的Spring项目来进行维护。
3. 在使用创建新的Spring项目时，建议使用自动装配及Java配置。

### 创建XML，声明Bean及构造器注入
**1)创建XML配置规范：**
1. JavaConfig的规范是@Configuration注解。而XML的规范是：
创建一个XML文件并且要以<beans>元素为根元素。
2. 基本的代码如下(引入了基本的，上下文的及aop的提示)：
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
			<!--其他配置细节-->
		</beans>
3. 装配Bean的最基本元素包含在spring-beans-3.1.xsd模式之中。
`<beans>`是该模式的一个元素，是所有Spring配置文件的根元素。
4. 此时的XML文件没有任何作用。因为还没有声明Bean。

**2)声明一个简单的Bean：**
1. xml的`<Beans>`元素相当于Java的@Configuration配置，而`<Bean>`元素则是相当于@Bean注解，用于声明简单的Bean。
2. 简单的声明方式如下：
		<bean class="chap2.SoftMusic"></bean> 
class属性用于指定要创建的是哪个类的Bean，默认的ID是类的全限定名`chap2.SoftMusic#0`,这个#0是计数器，如果再次声明一个相同的Bean,默认的ID就是`....#1`。当然很多时候需要手动去指定ID（或name，使用id属性即可）：
		<bean id="softMusic" class="chap2.SoftMusic"></bean>
因为将该Bean注入到另一个Bean中时指定ID更加方便而且不容易出错。
3. Xml声明Bean的注意事项：
Xml方式当有bean注解时调用构造器来创建bean的，而JavaConfig则会想尽一切办法来获取Bean。
Bean的类型以字符串的形式传给class属性，无法保证这个类存在或者不会修改，最好使用能够感知Spring功能的IDE(如STS)。

**3)构造注入初始化Bean简介：**
1. 在XML构造器注入初始化的Bean时，有很多种配置方案和风格选择。具体到构造器注入(还有一种Setter注入)，就有两种常用的方式：
	- `<constructor-arg>`元素
	- 使用Spring3.0所引入的c-命名空间
2. 前者`<constructor-arg>`比后者(c-命名空间)更麻烦但是功能更加完善。

**4)构造注入其他Bean：**
1. 需要装配的Bean的类的构造参数是引用了其他类型的对象的：注入bean对象。
2. 方式一：`<constructor-arg>`
		<bean id="softMusic" class="chap2.SoftMusic"></bean>   
		<bean id="musicplayer" class="chap2.MusicPlayer">
			<constructor-arg ref="softMusic"></constructor-arg>
		</bean>
没什么可说的，constructor-arg标签会告诉Spring要将一个id为softMusic的Bean引用传递到MusicPlayer的构造器中。
3. 方式二：使用一般的c-命名空间方式：
注意要修改上面的配置：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans
			xmlns="http://www.springframework.org/schema/beans"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:p="http://www.springframework.org/schema/p"
			<!--注意这里加了一行c-->
			xmlns:c="http://www.springframework.org/schema/c"
			xmlns:context="http://www.springframework.org/schema/context"
			xmlns:aop="http://www.springframework.org/schema/aop"
			xsi:schemaLocation="http://www.springframework.org/schema/beans
								http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
								http://www.springframework.org/schema/aop
								http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
								http://www.springframework.org/schema/context
								http://www.springframework.org/schema/context/spring-context-2.5.xsd">
			<bean id="softMusic" class="chap2.SoftMusic"></bean>
			<!--注入这样写-->
			<bean id="musicplayer" class="chap2.MusicPlayer" c:music-ref="softMusic"></bean>
		</beans>
重点在`c:music-ref="softMusic"`这个属性的写法，解释如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-07.jpg)
这是书上的图，我的beanID是softMusic。
由此看来使用c-命名空间的方式要比使用标签的方式简便的多。
4. c-命名空间的两种改进方式：(可能目前的新版本不支持这种写法了)
`c:music-ref="softMusic"`,但是这样也有一些问题，若果构造参数名字改变了(不再是music)，这样就行不通了，所以可以使用索引的方式标明这是该构造方法的第几个参数：
		<bean id="musicplayer" class="chap2.MusicPlayer" c:_0-ref="softMusic"></bean>
使用_0是因为不能以数字开头。
因为该构造方法只有一个参数，所以直接连数字都可以不用加：
		<bean id="musicplayer" class="chap2.MusicPlayer" c:_-ref="softMusic"></bean>

**5)将字面量注入到构造器中：**
1. 上面是注入Bean，这里是直接注入值。有时候就是直接使用值来注入来配置对象。
首先新建一个名为happyMusic的新实现类：
		package chap2;
		public class HappyMusic implements Music{
			private String song;
			private String singer;
			public HappyMusic(String song,String singer){
				this.song=song;
				this.singer=singer;
			}
			@Override
			public void play() {
				System.out.print("playing song "+song+" by singer "+singer);
			}
		}
Bean中这样写(c-命名空间的一般方式)：
		<bean id="happyMusic" class="chap2.HappyMusic" c:song="happy" c:singer="kenshine"></bean>
就不用在属性哪里加-ref参数了，因为不是引入其他bean。
然后再修改注入：
		<bean id="musicplayer" class="chap2.MusicPlayer" c:_-ref="happyMusic">
2. c-命名空间的改进方法(使用索引_n)：
		<bean id="happyMusic" class="chap2.HappyMusic" c:_0="happy" c:_1="kenshine"></bean>
但是此时就不能使用`c:_=""`的方式了，因为有两个参数而无法分辨。
3. 标签的方式：
		<bean id="happyMusic" class="chap2.happyMusic">
			<constructor-arg value="happy"></constructor-arg>
			<constructor-arg value="kenshine"></constructor-arg>
		</bean>
4. 测试类：
		public class MusicPlayerTest2 {
			//SystemRule库的一个Junit规则，可以将输出作为断言条件
			@Rule
			public final StandardOutputStreamLog log=new StandardOutputStreamLog();
			AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(MusicPlayer2Config.class);
			//使用返回值类型匹配
			private MediaPlayer player=context.getBean(MusicPlayer.class);
			//使用设置的name匹配
			private Music music=(Music)context.getBean("softMusic");
			@Test
			public void close(){
				context.close();
			}
			@Test
			public void musicNotBeNull(){
				Assert.assertNotNull(music);
			}
			@Test
			public void play(){
				player.play();
				//通过log.getLog()得到上面输出的值并进行断言测试
				Assert.assertEquals("playing song sad,singer : kenshine",log.getLog());
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-08.jpg)

**4）装配集合：-c命名空间的方式无法实现**
1. 创建normalMusic类：
		public class NormalMusic implements Music{
			private String song;
			private String singer;
			private List<String> tags;
			public NormalMusic(String song,String singer,List<String> tags){
				this.song=song;
				this.singer=singer;
				this.tags=tags;
			}
			@Override
			public void play() {
				System.out.println("playing song "+song+" by singer "+singer);
				for (String tag : tags) {
					System.out.print(tag+" ");
				}
			}
		}
最简单的配置(头尾略)：设为空`<null/>`标签
		<bean id="normanMusic" class="chap2.NormalMusic">
				<constructor-arg value="normal"></constructor-arg>
			<constructor-arg value="kenshine"></constructor-arg>
			<constructor-arg><null></null></constructor-arg>
		</bean>
		<bean id="musicPlayer" class="chap2.MusicPlayer">
			<constructor-arg ref="normanMusic"></constructor-arg>
		</bean>
这样子可行但是在调用play()方法时会报空指针异常。
2. 可选方案一：使用list标签搭配value子标签
		<bean id="normanMusic" class="chap2.NormalMusic">
			<constructor-arg value="normal"></constructor-arg>
			<constructor-arg value="kenshine"></constructor-arg>
			<constructor-arg>
				<list>
					<value>happy</value>
					<value>china</value>
					<value>popular</value>
					...
				</list>
			</constructor-arg>
		</bean>
可选方案二：使用list标签搭配ref导入tag类bean（假设有tag类）
		<bean id="normanMusic" class="chap2.NormalMusic">
			<constructor-arg value="normal"></constructor-arg>
			<constructor-arg value="kenshine"></constructor-arg>
			<constructor-arg>
				<list>
					<ref bean="tag1"/>
					<ref bean="tag2"/>
					<ref bean="tag3"/>
					...
				</list>
			</constructor-arg>
		</bean>
可选方案三：使用set标签搭配value或者ref标签：
		<bean id="normanMusic" class="chap2.NormalMusic">
			<constructor-arg value="normal"></constructor-arg>
			<constructor-arg value="kenshine"></constructor-arg>
			<constructor-arg>
				<set>
					<value>happy</value>
					<value>china</value>
					<value>popular</value>
					...
				</set>
			</constructor-arg>
		</bean>
`<set>`标签与`<list>`标签的作用差不多，只不过前者去重而后者不去重。两者都可以用来装配list或者set类型的数据。
3. map类型：
		<bean id="normalMusic" class="chap2.NormalMusic">
			<constructor-arg>
				<map>
					<entry key="k1" value="v1"/>
					<entry key="k2" value="v2"/>
					<entry key="k3" value="v3"/>
					...
				</map>
			</constructor-arg>
		</bean>
更多详细的配置可以参考博客：
[Spring中常用类型的bean配置(Map,List,Set,基本类型)](https://www.cnblogs.com/chyu/p/5207994.html)

### Setter注入Bean（Setter方法）
**1)Setter注入简介及选择：**
1. 创建MusicPlayer2类如下：
		public class MusicPlayer2 implements MediaPlayer{
			private Music music;
			//一定要有一个空的默认方法
			public MusicPlayer2(){
			}
			//构造器注入
			public MusicPlayer2(Music music){
				this.music=music;
			}
			//set方法注入
			public void setMusic(Music music) {
				this.music = music;
			}
			@Override
			public void play() {
				music.play();
			}
		}
2. 如何选择构造器注入或者Setter注入呢？(常用但不强制)
强依赖选择构造器：如音乐的曲目，歌手等，必须要存在的属性。
弱依赖选择Setter：如音乐播放器的音乐，即使没有但也影响不大。

**2)简单Bean注入的配置：**
1. 最简单的配置(头尾略)：
		<bean id="softMusic" class="chap2.SoftMusic"></bean>
		<bean id="musicPlayer2" class="chap2.MusicPlayer2">
			<property name="music" ref="softMusic"></property>
		</bean>
`<property>`属性为Setter方法提供的功能和`<constructor-arg>`属性为构造器所提供的功能一样。
而构造器有c-命名空间的方法简化配置，相应的，属性注入也有p-命名空间的方法简化配置。
2. 使用p-命名空间：
需要先导入配置：在Beans标签内加上这个配置
		xmlns:p="http://www.springframework.org/schema/p"
xml的配置可以写成这样：
		<bean id="softMusic" class="chap2.SoftMusic"></bean>
		<bean id="musicPlayer2" class="chap2.MusicPlayer2" p:music-ref="softMusic">
		</bean>
其它p-命名空间所遵循的规范和c-命名空间相同。(包括索引或者只写`_`等)
3. 书上对p-命名空间各部分的介绍图：(和c-命名空间不一样)
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-09.jpg)

**3)Setter注入字面量及集合类：**
1. 首先修改normalMusic类如下：
		public class NormalMusic implements Music{
			private String song;
			private String singer;
			private List<String> tags;
			//切记一定要有一个空的构造方法
			public NormalMusic(){
			}
			public NormalMusic(String song,String singer,List<String> tags){
				this.song=song;
				this.singer=singer;
				this.tags=tags;
			}
			public void setSong(String song) {
				this.song = song;
			}
			public void setSinger(String singer) {
				this.singer = singer;
			}
			public void setTags(List<String> tags) {
				this.tags = tags;
			}
			@Override
			public void play() {
				System.out.println("playing song "+song+" by singer "+singer);
				for (String tag : tags) {
					System.out.print(tag+" ");
				}
			}
		}
2. 配置类代码如下：
		<bean id="normalMusic" class="chap2.NormalMusic">
			<property name="song" value="good"></property>
			<property name="singer" value="kenshine"></property>
			<property name="tags">
				<list>
					<value>china</value>
					<value>popular</value>
					<value>666</value>
				</list>
			</property>
		</bean>
		<bean id="musicPlayer2" class="chap2.MusicPlayer2" p:music-ref="normalMusic">
		</bean>
这些配置和构造器的配置非常相似，集合类的注意事项也相似。就不多写了。
3. 也可以使用p-命名空间，但是集合类的无法配置：
		<bean id="normalMusic" class="chap2.NormalMusic"
		p:song="good" p:singer="kenshine">
			<!--集合类不能使用p-命名空间装配-->
			<property name="tags">
				<list>
					<value>china</value>
					<value>popular</value>
					<value>666</value>
				</list>
			</property>
		</bean>
		<bean id="musicPlayer2" class="chap2.MusicPlayer2" p:music-ref="normalMusic">
		</bean>
p-命名空间和c-命名空间一样，有-ref是装配Bean,没有则是装配字面量。

**4)使用Spring util-命名空间简化集合类配置：**
1. 需要在配置文件顶部添加如下配置：
		xmlns:util="http://www.springframework.org/schema/util" 
如果要添加有注释则需要额外的配置，如下：
		<beans  
			xmlns="http://www.springframework.org/schema/beans"  
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:p="http://www.springframework.org/schema/p"  
			xmlns:c="http://www.springframework.org/schema/c"
		 	xmlns:util="http://www.springframework.org/schema/util"
			xmlns:context="http://www.springframework.org/schema/context"
			xmlns:aop="http://www.springframework.org/schema/aop"
			xsi:schemaLocation="http://www.springframework.org/schema/beans
								http://www.springframework.org/schema/beans/spring-beans.xsd
								http://www.springframework.org/schema/aop
								http://www.springframework.org/schema/aop/spring-aop.xsd
								http://www.springframework.org/schema/context
								http://www.springframework.org/schema/context/spring-context.xsd
								http://www.springframework.org/schema/util
								http://www.springframework.org/schema/util/spring-util.xsd"> 
2. util-命名空间可以将集合类单独封装成一个bean,然后在装配时只要导入这个util的id就可以了，简单使用如下：
		<util:list id="taglist">
			<value>china</value>
			<value>popular</value>
			<value>666</value>
		</util:list>
		<bean id="normalMusic" class="chap2.NormalMusic"
		 p:song="good" p:singer="kenshine" p:tags-ref="taglist">
		</bean>
		<bean id="musicPlayer2" class="chap2.MusicPlayer2" p:music-ref="normalMusic">
		</bean>
3. list只是util的一种命名空间，用于生成java.util.List类型的Bean。
util还有以下几种类型的命名空间：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-10.jpg)
4. 创建测试代码：
		public class MusicPlayerTest3 {
			//SystemRule库的一个Junit规则，可以将输出作为断言条件
			@Rule
			public final StandardOutputStreamLog log=new StandardOutputStreamLog();
			ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("/chap2/musicplayer3.xml");
			//使用返回值类型匹配
			private MediaPlayer player=context.getBean(MusicPlayer2.class);
			//使用设置的name匹配
			private Music music=context.getBean(Music.class);
			@Test
			public void close(){
				context.close();
			}
			@Test
			public void musicNotBeNull(){
				Assert.assertNotNull(music);
			}
			@Test
			public void play(){
				player.play();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-11.jpg)

**5)根据我的理解画的图**
1. 看图：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-12.jpg)
2. 其中Music的实现类到MusicPlayer的注入是通过id来配置的。
util-命名空间产生的bean也是通过配置id来实现注入的。

---
## 5.导入和混合配置
**1)简单理论：**
1. 混合配置，就是混合自动，JavaConfig和XML的配置。
2. 自动装配时，并不在意要装配的bean来自哪里，自动装配的时会考虑到Spring容器中的所有bean,不管它是从何种配置得到的。

**2)在JavaConfig中引用XML配置：**
1. 假设有两个音乐类需要配置：(oldMusic和newMusic)
		public class OldMusic implements Music{
			@Override
			public void play() {
				System.out.println("this is oldmusic");
			}
		} 
		public class NewMusic implements Music{
			@Override
			public void play() {
				System.out.println("this is newmusic");
			}
		}
2. 一起配置的Config：
		@Configuration
		public class NewOldConfig {
			@Bean
			public Music newmusic(){
				return new NewMusic();
			}
			@Bean
			public Music oldmusic(){
				return new OldMusic();
			}
		}
3. 但是想让这俩实现类在不同的配置类，有两种方案(不全写代码了)：
方案一：只要将一个导入另一个即可
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-13.jpg)
		@Configuration
		@import(OldMusicConfig.class)
		public class NewMusicConfig {
			@Bean
			public Music newmusic(){
				return new NewMusic();
			}
		}
方案二：更好的方案
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-14.jpg)
		@Configuration
		@Import({OldMusicConfig.class，NewMusicConfig.class})
		public class SystemConfig {
		}
使用@import标签可以引入其他的java配置。
4. 如果有需求明确OldMusic需要用XML配置而NewMusic需要用Java配置，则SystemConfig配置如下：
		@Configuration
		@Import(NewMusicConfig.class)
		@ImportResource("classpath:oldconfig.xml")
		public class SystemConfig {
		}
简单的示意图为：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-15.jpg)
在JavaConfig配置类中使用@ImportResource("xml路径")就可以引入xml的配置了。

**3)XML配置中导入JavaConfig配置：**
1. 两个XML文件之间的相互导入：`<import>`标签的Resource属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-16-1.jpg)
两个小的XML配置如下:
NewMusicConfig.xml：
		<bean id="oldMusic" class="chap2.OldMusic"></bean> 
OldMusicConfig.xml：
		<bean id="newMusic" class="chap2.NewMusic"></bean>
MusicConfig.xml:
		<import resource="OldMusicConfig.xml"/>
		<import resource="NewMusicConfig.xml"/>
使用<import>标签的resource属性能达到和@importResource注解的效果。
2. 如果其中一个是JavaConfig配置的(假设为NewMusic),则MusicConfig.xml中这样配置(不需要额外的标签)：
		<import resource="OldMusicConfig.xml"/>
		<import class="NewMusicConfig.class"/>
![](http://p5ki4lhmo.bkt.clouddn.com/00040Spring%E5%AD%A6%E4%B9%A02-17.jpg)
使用<import>标签的class属性能达到和@import注解的效果。

---