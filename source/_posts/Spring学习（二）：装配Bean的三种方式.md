---
title: Spring学习（二）：装配Bean的三种方式
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
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MusicPlayer2Config.class)
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



---