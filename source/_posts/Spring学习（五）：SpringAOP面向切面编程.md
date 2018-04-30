---
title: Spring学习（五）：SpringAOP面向切面编程简介
date: 2018-4-30 11:07:08  
tags:
- Spring
- Spring核心框架
- Spring AOP
categories: J2EE框架

---
## 0.学习准备
1. 参考书籍：《Spring实战》(第四版)

2. 内容概述：
	- AOP常用术语
	- 面向切面编程的基本原理
	- 通过POJO创建切面
	- 使用@AspectJ注解
	- 为AspectJ切面注入依赖

---
## 1.AOP术语
**1)AOP术语：**
1. 横切关注点：
散布在应用中多处的功能称为横切关注点，这些横切关注点从概念上是与业务逻辑相分离的，但是往往又会直接嵌入到应用的业务逻辑之中。(如日志，事务，安全等功能)
而切面(Aspect)则实现了横切关注点的模块化：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-01.jpg)
这些横切关注点被模块化为特殊的类，称为切面。
2. 切面是代码重用的又一种可选方案：
以前都是通过继承(inheritance)和委托来实现的。
如果在一个应用中都继承一个基类，那么会导致一个脆弱的对象体系。
使用委托则需要对委托的对象进行复杂的调用。
面向切面的编程(AOP)在更多场景下代码更加清晰简洁。
使用切面的好处：
	- 服务模块更加简洁
	- 每个关注点都集中在一个地方
3. 通知(Advice)：
切面的工作被称为通知，通知定义了切面是什么以及何时使用。(如何执行，何时执行)
Spring切面可以应用5种类型的通知：
	- 前置通知：在目标方法被调用之前调用通知的功能
	- 后置通知：在目标方法被调用之后调用通知的功能(不用管该方法的输出是什么)
	- 返回通知：在目标方法成功执行之后调用通知的功能
	- 异常通知：在目标方法抛出异常之后调用通知
	- 环绕通知：通知包裹了被通知的目标方法，在目标方法被执行之前和之后都执行自定义的行为
4. 连接点(Join point)：
连接点是程序执行过程中能够插入(织入)切面的一个点。
(切点前和切点后，但远不止这两种)
5. 切点(Pointcut)：
切面定义了如何执行与何时执行，而切点则定义了在何处执行。
通常使用明确的类和方法或者利用这则表达式来匹配类和方法名来指定这些切点。
有些AOP框架允许创建动态切点。
6. 切面(Aspect)：
横切关注点被模块化后的特殊的类。
通知和切点定义了一个切面的所有要素：怎样执行，何时执行，何处执行。
7. 引入(Introduction)：
引入允许我们向现有的类添加新方法或者属性。
可以在不修改这些现有类的情况下，让他们具有新的行为和状态。
8. 织入(Weaving)：
切面在指定的连接点被织入到目标对象中。
织入是把切面应用到目标对象并创建新的代理对象的过程。
目标对象生命周期中有多个点可以进行织入：
	- 编译期：需要特殊的编译器，如AspectJ的织入编译器
	- 类加载期：切面在目标被加载到JVM时被织入，需要特使的类加载器。
	AspectJ5的加载时织入。
	- 运行期：切面在应用运行的某个时刻被织入。Spring AOP就是这种方式。
9. 简单图示：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-02.jpg)

---
## 2.Spring对AOP的支持
1. AOP框架的基本功能：
创建切点来定义切面所织入的连接点。
2. Spring提供了4种类型的AOP支持：
	- 基于代理的经典Spring AOP(非常笨重但是很经典)
	- 纯POJO切面(XML配置，使用aop命名空间)
	- @AspectJ注解驱动的切面(可以不用XML配置)
	- 注入式AspectJ切面(适用于Spring的各版本)
3. Spring通知是Java编写的：
Spring所有的通知都是用标志的Java类编写的，有助于我们学习和开发。
AspectJ是通过Java语言扩展的方式来实现的，通过特有的AOP语言可以获得更强大和细粒度的控制，以及更加丰富的AOP工具。但是我们需要额外学习新的工具和语法。
4. Spring在运行时通知对象(进行织入)。
5. Spring是基于动态代理的，所以只支持方法级别的连接点。(无法创建细粒度的通知)
其他的AOP框架如AspectJ和JBoss则提供了字段和构造器的接入点。

---
## 3.通过切点来选择连接点
**1)简介：**
1. 在Spring AOP中，要使用AspectJ切点表达式语言来定义切点。
AspectJ主要通过指示器来执行与限制匹配。
2. Spring支持的AspectJ指示器：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-03-1.jpg)
使用其他的指示器时会抛异常。
3. 只有execution()指示器是用来执行匹配的，其他都是用来限制匹配的。

**2)创建切点：**
1. 简单的创建示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-04.jpg)
2. 对应的接口如下：
		package concert;
		public interface Performance{
			public void perform();
		}
3. 如果需要在concert包下匹配，则需要使用within()指示器：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-05.jpg)
两个指示器之间的关系有：&&(and),||(or),!(not)
4. 因为&在XML中有特殊的含义，所以建议使用英文(and,or,not)来代替符号。

**3)在切点中选择特定的Bean：**
1. Spring除了上述的AspectJ指示器外，还引入了一个新的Bean()指示器。
可以使用BeanID或者Bean名称作为参数来限制节点值匹配特定的Bean。
2. 如：
		excute(* concert.Performance.perform() and bean('goodMovie'));
只有ID和goodMovie相等的Bean才会编织对应的通知。
3. 或者可以让除某个ID以外的bean都进行编织：
		excute(* concert.Performance.perform() and bean('badMovie'));

---
## 4.使用注解创建切面
使用注解来创建切面是AspectJ5引入的新特性。
AspectJ面向注解的模型可以非常便捷地通过少量注解把任意类转变为切面。
在pom.xml中配置依赖导包：(除了Spring所有包之外还需要添加如下的包)

		<!-- Spring AOP需要的依赖包 -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>1.8.9</version>
		</dependency>
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>1.9.0</version>
		</dependency>
		<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>3.2.6</version>
		</dependency>

### 定义切面
**1)一般方式定义切面：**
1. 创建表演接口如下：(纵向业务)
		package chap4;
		//表演接口
		public interface Performance {
			public void perform();
		}
2. 表演实现类如下：
		@Component
		public class MagicPerformance implements Performance {
			@Override
			public void perform() {
				System.out.println("the magic performance is playing");
			}
		}
3. 创建监视类Monitor来监控这次表演：(横向切面)
		package chap4;
		import org.aspectj.lang.annotation.*
		//监控类
		@Component
		@Aspect
		public class Monitor {
			//表演之前
			@Before("execution(** chap4.Performance.perform(..))")
			public void performanceBegin(){
				System.out.println("performance begin!");
			}
			//表演之后，无论成功与否
			@After("execution(** chap4.Performance.perform(..))")
			public void performanceAfter(){
				System.out.println("performance end!");
			}
			//表演之后(成功)
			@AfterReturning("execution(** chap4.Performance.perform(..))")
			public void AfterSuccessPerformance(){
				System.out.println("the performance is great!");
			}
			//表演之后(失败:报异常)
			@AfterThrowing("execution(** chap4.Performance.perform(..))")
			public void AfterUnSuccessPerformance(){
				System.out.println("the performance is awful!");
			}
		}
3. 各个注解的作用：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-06.jpg)

**2)通过切点的方式定义切面：**
1. 可以发现上面的相同的AspectJ表达式语句重复了4次：
		execution(** chap4.Performance.perform(..))
2. 可以通过@Pointcut方法来声明一个切点：上述代码可以修改如下
		@Component
		@Aspect
		public class Monitor {
			@Pointcut("execution(** chap4.Performance.perform(..))")
			public void perform(){}
			//表演之前
			@Before("perform()")
			public void performanceBegin(){
				System.out.println("performance begin!");
			}
			//表演之后，无论成功与否
			@After("perform()")
			...
			//表演之后(成功)
			@AfterReturning("perform()")
			...
			//表演之后(失败:报异常)
			@AfterThrowing("perform()")
			...
		}

**3)配置文件的写法及测试：**
1. MagicPerformanceConfig代码如下：
		@Configuration
		@EnableAspectJAutoProxy		//启用AspectJ自动代理
		@ComponentScan(basePackageClasses=Performance.class)
		public class MagicPerformanceConfig {
		}
注意要加上@EnableAspectJAutoProxy注解。
其实是需要声明Moniter切面的bean的：
		@Bean
		public Moniter moniter(){
			return new Moniter();
		}
因为Moniter使用了@Companent注解可以自动装配Bean。
2. MagicPerformance.xml配置文件中代码如下：
		<context:component-scan base-package="chap4"></context:component-scan>
		<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
		<bean class="chap4.Monitor"></bean>
其中那个Bean配置了自动扫描也可以省略。
3. 测试类：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MagicPerformanceConfig.class)
		public class MagicPerformanceTest {
			@Autowired
			@Qualifier("magicPerformance")
			private Performance performance;
			
			@Test
			public void testAop(){
				performance.perform();
			}
		}
4. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-07.jpg)

**4)总结以及一些补充：**
1. Spring自动代理(无论是JavaConfig方式还是XML方式)仅仅是@Aspect作为创建切面的指导，切面依然是基于代理的。
2. 要想利用AsoectJ的所有功能，必须在运行时使用AspectJ并且不依赖Spring来创建基于代理的切面。

### 环绕通知与引入新功能
**1)环绕通知创建与使用：**
1. 环绕通知是最强大的通知类型，能够让你所编写的逻辑将被通知的目标方法完全包装起来。(可以用来实现其他几种通知)
2. 上面的Moniter可以修改如下：
		@Component
		@Aspect
		public class Monitor {
			@Pointcut("execution(** chap4.Performance.perform(..))")
			public void perform(){}
			
			@Around("perform()")
			public void AroundPerform(ProceedingJoinPoint joinPoint){
				try{
					System.out.println("performance begin!");
					//调用perform方法
					joinPoint.proceed();
					System.out.println("performance end!");
					System.out.println("the performance is great!");
				}catch(Throwable e){
					System.out.println("the performance is awful!");
				}
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-08.jpg)
3. 这个通知方法必须接受一个ProceedingJoinPoint接口对象。
可以通过该对象的proceed()方法来调用目标对象的方法(将控制权交给被通知的方法)
4. 可以不使用proceed()来阻塞对被通知方法(切点)的访问。
也可以多次使用proceed()来实现被通知方法的多次调用。

**2)通过注解引入新功能：**
1. 一些动态语言如Ruby等有开放类的概念，他们可以不直接修改地偶像或者类的定义就能够为对象或类添加新的方法。但是Java不是动态语言，一旦编译完成，就很难添加新的功能。但是可以通过被引入的AOP来为Spring Bean添加新的方法。
2. 如要为前面的MagicPerformance添加一个彩排的方法：
创建彩排(准备表演)接口如下：
		public interface Prepare {
			public void preparePeform();
		}
彩排实现类：
		public class PrepareMagic implements Prepare{
			@Override
			public void preparePeform() {
				System.out.println("preparing Magic performance!");
			}
		}
引入通知类：(重点就是这个)
		@Aspect
		@Component
		public class PrepareAddInMagic {
			@DeclareParents(value="chap4.Performance+",defaultImpl=PrepareMagic.class)	//注意一定要带上包名
			public Prepare prepare;		//可以加个static也可以不加
		}
将javaConfig组件扫描的范围去除。
测试代码如下：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=MagicPerformanceConfig.class)
		public class MagicPerformanceTest {
			@Autowired
			@Qualifier("magicPerformance")
			private Performance performance;
			
			@Test
			public void testAop(){
				Prepare prepare=(Prepare)performance;
				prepare.preparePeform();
				performance.perform();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-09.jpg)
3. 需要注意的是performance使用引入对象时的用法。
4. 关于@DeclareParents的说明：
	- `value`：指定那种类型的bean需要引入该接口。记得一定要带包名，最后的+代表该类的所有实现类都添加这个方法。
	- `defaultImpl`：指定为引用功能提供的实现类。(这里就是PrepareMagic类)
	- `@DeclareParents`标注的属性指明了要引入的接口。
5. 更简单的例子:
<https://www.cnblogs.com/xxdsan/p/6496332.html>
6. 简单图示：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-10.jpg)

### 处理带参数的通知
**1)简单的准备工作：**
1. 创建接口Performance2：
		public interface Performance2 {
			public void perform(String name);
		}
2. 创建DancePerformance实现类：
		@Component
		public class DancePerformance implements Performance2 {
			@Override
			public void perform(String name) {
				System.out.println("dancer"+name+"is performing");
			}
		}

**2)创建切面并测试：**
1. 创建监视类：
		@Aspect
		@Component
		public class DanceMoniter {
			@Pointcut("execution(** chap4.Performance2.perform(String)) && args(name)")
			public void perform(String name){}
			
		//	@Before("perform(name)")
		//	public void BeforePerform(String name){
		//		System.out.println(name+" begin dance");
		//	}
			
			@Around("perform(name)")
			public void AroundPerform(ProceedingJoinPoint joinPoint,String name){
				try{
					System.out.println("performance begin!");
					joinPoint.proceed(new Object[]{name});
					System.out.println("performance end!");
					System.out.println("the performance is great!");
				}catch(Throwable e){
					System.out.println("the performance is awful!");
				}
			}
			
		}
2. 创建JavaConfig配置类DancePerformanceConfig：
		@Configuration
		@EnableAspectJAutoProxy
		@ComponentScan
		public class DancePerformanceConfig {
		}
3. 测试类：
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes=DancePerformanceConfig.class)
		public class DancePerformanceTest{
			@Autowired
			@Qualifier("dancePerformance")
			private Performance2 performance2;
			
			@Test
			public void testAop(){
				performance2.perform("kenshine");
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00054Spring%E5%AD%A6%E4%B9%A05-11.jpg)

**3)简单总结：**
1. 使用@Around注解时，在方法体内的joinPoint.proceed需要像上面那样写。
`joinPoint.proceed(new Object[]{name});`,name则是使用args传进来的参数。
2. 使用args传入的参数可以在该通知类中的任意一个地方使用。
3. 使用其他注解如@Before等则没有像@Around的那种限制。
4. 这种方法配置切面很方便，但是唯一的缺点就是需要知道通知类的源码。如果不知道源码，那么我们就需要另外一种配置方式：XML声明切面

---
## 5.在XML中声明切面


---