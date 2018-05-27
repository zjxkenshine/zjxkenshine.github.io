---
title: SpringMVC学习（六）：EL表达式及国际化
date: 2018-5-26 21:35:02  
tags: 
- Spring
- Spring MVC
- JSTL&EL
categories: Java框架

---
## 0.学习准备
1. 参考资料：
参考书籍《Spring MVC学习指南》(第二版)(基础学习)
《Spring实战》(第四版)(综合运用)
参考视频：
尚硅谷《Spring MVC》(补充知识)
2. 简单目录：
	- EL表达式语言简介
	- EL表达式语法
	- EL隐式对象(内置对象)
	- EL运算符
	- 引用静态属性和静态方法
	- 集合的创建与操作
	- EL的格式化操作
		- 格式化集合
		- 格式化数据
		- 格式化时间
	- JSP2.0以上EL的配置
	- 国际化及语言区域简介
	- 国际化SpringMVC
	- message标签
	- 国际化示例

---
## 1.EL表达式简介
1. JSP2.0最重要的特性之一就是表达式语言(EL),
JSP可以通过EL表达式来访问应用程序的数据。
2. JSP2.0最初是将EL和JSTL绑定到一起的，后来就不需要了，可以单独在JSP页面中使用EL表达式。
3. 2013年5月发布了EL3.0(JSR 349)，EL不再是JSP或其他技术的一部分，而是一个独立的规范。
4. EL3.0增加了对lambda表达式的支持，并新增了集合操作，而且不需要Java8的支持，只需要Java7就可以了。
5. java自带了`javax.el`包，所以不需要再导包了。
6. 目前如Tomcat 8、Jetty 9、GlasshFish 4已经支持EL 3.0。新特性包括：如字符串拼接操作符、赋值、分号操作符、对象方法调用、Lambda表达式、静态字段/方法调用、构造器调用、Java8集合操作。目前Glassfish 4/Jetty实现最好，对大多数新特性都支持、Tomcat8对如Lambda、静态字段/方法调用、构造器调用目前不支持

---
## 2.表达式语言的语法
**1)语法简介：**
1. 有以下两种使用方式：
`${expression}`
`#{expression}`
2. `#{exp}`和`${exp}`都是有EL引擎以相同的方式进行计算。
在未被当做独立的引擎而是作为JSP底层技术时有如下不同：
	- `${exp}`用于立即计算，与JSP页面一同编译执行。
	- `#{exp}`用于延迟计算，只有要使用该值时才会进行计算。
	- 在JSP2.1及更高版本中`#{exp}`形式只能在接收延迟值的属性中才能使用。
3. 如果在定制标签的某一属性中使用EL表达式，那么该表达式的取值结果字符串会强制编程该属性需要的类别。

**2)关键字：**
以下是EL表达式的关键字，不能作为标识符：
		and eq gt true instanceof or ne le
		false empty not ge null div mod

**3)[]与.运算符：**
1. 两种取值方式：
		${对象名["属性名"]}
		${对象名.属性名}
使用`[]`更加规范，而使用小数点`.`则更加简洁。
2. 如要取得`pageContext.request.servletPath`这个值可以有如下四种取法：
		${pageContext.request.servletPath}
		${pageContext.request["servletPath"]}
		${pageContext["request"].servletPath}
		${pageContext["request"]["servletPath"]}

**4)取值规则：**
1. 如有型如`${expr-a[expr-b]}`，则该EL表达式的取值规则如下：(不用细看）
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-01.jpg)
2. 简单来说就是：
任意为null返回null,非空匹配上则返回有效值，否则返回null。

**5)访问JavaBean：**
1. 可以通过如下两种方式访问JavaBean中的属性值：
		${bean名称["属性名称"]}
		${bean名称.属性名称}
2. 如果bean的某个属性是一个集合类，则可以使用多次`[]`或者`.`来获取集合类中的值。

---
## 3.EL隐式对象（内置对象）
隐式对象，就是可以不显示使用，但是可以使用的对象。

**1)EL隐式对象简介：**
1. 在JSP页面中可以通过JSP脚本来访问JSP内置对象。
EL则提供了一组自己隐式对象来访问不同的值。
2. EL隐式对象表如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-02.jpg)
3. 下面对这些内置对象一一介绍。

**2)pageContext对象：**
1. 表示当前JSP的javax.servlet.jsp.PageContext。
包含了所有JSP的内置对象。
2. JSP内置对象如下表：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-03.jpg)
3. 如可以使用`${pageContext.request.property}`来访问请求对象的属性,请求对象的属性如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-04.jpg)
4. 因为请求参数非常常用，所以有几个对象专门处理Request请求。

**3)initParam对象：**
1. 用于获取上下文参数的值。
2. 示例：获取上下文中的name参数值
`${initParam.name}`
或者是`[]`

**4)param对象：**
1. 用于获取请求参数的映射(Map)。
2. 使用param就可以获得Map,然后使用`.`或者`[]`来获取指定参数的值。
3. 如获取请求参数name的值：
`${param.name}`
或者：
`${param["name"]}`

**5)paramValues对象：**
1. 返回的是一个`Map<参数名，值数组>`，用于获取一个请求参数的所有值。
只有一个值也是返回数组。
2. 示例：
`${paramValues.name[0]}`
`${paramValues.name[1]}`

**6)header对象：**
1. 用于获取所有请求标题的Map。(`Map<标题名称，标题只>`)
2. 关于请求标题(请求头)：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-05.jpg)
除了首位两行其余都是请求标题。(详见SpringMVC学习笔记一)
3. 如要获取Connection的值：
`${header.["Connection”]}`
如果该名称符合Java变量名规范也以使用：
`${header.Connection}`

**6)headerValues对象：**
1. 与paramValues对象类似，返回的是`Map<header名称，值数组>`
2. 如取Connection的值：
`${headerValues.["Connection”][0]}`
`${headerValues.["Connection”][1]}`

**7)cookie对象：**
1. 用来获取一个Cookie，表示当前HttpServletRequest中有Cookie值。
2. 获取名为userid的Cookie的值：
`${cookie.userid.value}`
获取名为userid的Cookie的路径：
`${cookie.userid.path}`

**8)XXXScope对象：**
1. 有applicationScope，sessionScope，requestScope，PageScope
2. 表示获取不同范围级别的变量的值。
3. 可以省略这些对象，默认获得的是最小级别的变量的值。
(会依次在pageContext，servletRequest，HttpServlet和ServletContext中寻找)
4. 如application,session,request和page中都有name变量，
使用`${name}`获得的是page级别的变量。

---
## 4.EL运算符
**1)运算符简介：**
1. 除了`.`和`[]`运算符之外，EL还提供了算术、关系、条件、逻辑，Empty运算符，字符串连接运算符以及分号连接运算符。
2. 由于EL是为J方便SP免脚本编程，所以除了关系条件运算符其余运算符的使用范围都很有限。

**2)算术运算符：**
1. 有以下运算符：
`+,-,*,/,div,%,mod`
2. 优先级：
高：`*,/,div,%,mode`
低：`+-`

**3)关系运算符：**
- 等于：`==`或`eq`
- 不等于：`!=`或`ne`
- 大于：`>`或`gt`
- 小于：`<`或`lt`
- 大于等于：`>=`或`ge`
- 小于等于：`<=`或`le`

**4)逻辑运算符：**
- 与：`&&`或`and`
- 或：`||`或`or`
- 非：`!`或`not`

**5)条件运算符：**
与Java中的三目运算符类似：
`${条件?true:false}`

**6)empty运算符：**
1. 用来检查是否为null或者empty：
`${empty X}`
2. 可以检查长度为0的字符串，空Map，空数组和空集合。
如果是空则返回True。

**7)字符串连接运算符与分隔运算符：**
1. `+=`可以用来连接字符串。
2. 分号`:`可用来分隔两个表达式。

---
## 5.引用静态字段和静态方法
1. 可以引用Java类的静态字段及静态方法。
需要先使用page伪指令来导入包或者类。(`java.lang.*`包自动导入)
2. 使用page指令导包：
		<%@ page import="包或类的路径" %>
3. 使用方式：
`${Math.PI}`
4. 还有一种导包方式：(通过实现监听器导包)
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-06.jpg)

---
## 6.创建操作集合，Map
**1)创建Set，List和Map：**
1. 创建Set：
语法：(花括号)
`${{Set中的元素}}`
示例：创建一个简单的集合
`${{1,2,3,4,5,6,7}}`
2. 创建List：
语法：(中括号)
`${[List中的元素]}`
示例：
`${[1,2,3,4,5]}`
3. 创建Map：
语法：
`${{k1:v1,k2:v2...}}`

**2)访问列表或者Map：**
1. 访问列表元素：(使用JSTL)
		<c:set var="numlist" value="${[1,2,3,4,5]}">
		<c:out>${numlist[0]}</c:out>
2. 访问Map元素：(不使用JSTL)
		${map={"k1":1,"k2":2}}"
		${map["k1"]}

**3)操作集合：流简介**
1. EL3.0的新特性之一就是操作集合，可以通过流方法将集合转换为流来使用各种流操作。
2. stream()方法将集合转换为流：
如一个java.util.List类的mylist转换为流：
`${mylist.stream()}`
3. 大部分流操作会返回一个流，所以可以形成链式操作：
`${mylist.stream().流操作1.流操作2.toList()}`
4. toList()方法可以将流转换为List类型，方便打印或者格式化结果。
5. toArray()方法将流转换为Java数组。

**4)其余流方法简介：**
- limit()方法：
限制流中元素的数量
		${[a,b,c,d,e,f,g].stream().limit(3).toList()}
最后获得的List如下：
`[a,b,c]`
- sort()方法：
对元素进行排序
		${mylist.stream().sort().toList()}
- average()方法：
返回流中所有元素的平均值
返回值是一个Optional对象，可能为null，需要调用get()获得实际值。
		${mylist.stream().average().get()}
- sum()方法：
计算流中所有元素的总和
		${[1,5,7,5].stream().sum()}
- count()方法：
计算流中元素的数量
		${mylist.stream().count()}
- min()/max()方法：
返回流元素中的最小，最大值，返回值是一个Optinal对象，需要用get()获取
		${mylist.stream().min().get()}
		${mylist.stream().max().get()}
- map()方法：
将流中的每一个元素映射到另一个流中的另一个元素，并返回新流。
接受一个lambda表达式做参数。
如`map(x->2*x)`表示将每个元素都乘2，`map(x->x.toUpperCase())`表示每个元素都转换为大写：
		${[2,3,4].stream().map(x->2*x).toList()}
		${[a,b,c].stream().map(x->x.toUpperCase()).toList()}
- filter()方法：
接受lambda表达式做参数，根据表达式过滤流元素，返回过滤结果组成的流。
测试是否以某个字母开头：`filter(x->x.startsWith("s"))`
		${namelist.stream().filter(x->x.startsWith("s")).toList()}
- forEach()方法：
也是以lamda表达式做参数，对流中的所有元素做操作，返回空。
如将某一列表的所有值打印：
		${namelist.stream().forEach(x->system.out.println(x))}

---
## 7.EL格式化操作(集合，数字，日期)
**1)EL格式化简介：**
1. EL主要定义了如何写表达式而不是函数，所以无法直接打印或者格式化集合。
打印和格式化主要是交给JSTL来做，但是在EL3.0中已经可以实现打印和格式化集合了。
2. EL3.0允许我们完全抛弃JSTL,虽然一般不会那么做。
3. 如上面的forEach()方法可以用来遍历
但是需要对元素做一定的格式化草能进行输出
4. 有两种方式可以对集合进行格式化。

**2)格式化集合：使用HTML注解**
1. 适用于Java7及以上版本。
2. 需要将列表：
`[elem1,elem2,elem2..]`
输出为如下格式
		<ul>
			<li>elem1</li>
			<li>elem2</li>
			<li>elem3</li>
			...
		<ul>
3. 可以使用如下的方式进行：(使用HTML注释)
		...
		${cities=[北京,天津,上海,杭州]}
		<ul>
		<!-- 
		${cities.stream().map(a->"--><li>"+=a+="</li><!--").toList() }
		-->
		</ul>
		...
解析后的代码如下：
		<ul>
		<!--[--><li>北京</li><!--,-->
		<li>天津</li><!--,-->
		<li>上海</li><!--,-->
		<li>杭州</li><!--]-->
		</ul>
4. 我的Tomcat8好像不支持EL3.0的部分功能，所以就不测试了。

**3)格式化集合：使用String.join()方法**
1. 格式化的第二个方法，需要引用静态字段及方法。
只对Java8及以上有效。
2. Java8中String新增了静态方法join：
		public static String join(CharSequence delimiter,Iterable<? extends CharSequence> elements) {
		...
		}
 		public static String join(CharSequence delimiter, CharSequence... elements) {
		...
		}
该方法的作用简单来说就是用指定分隔符delimiter连接循环elements中的元素，并且返回一个字符串。
3. 所以上面的处理可以写成下面的形式：
		${empty cities?"":"<ul><li>"+=String.join("</li><li>",cities)+="</li><ol>"}

**4)格式化数字：**
1. 也需要使用EL3.0引用静态方法的特性。
String.format()方法可以用来格式化数字。
2. format方法的源码如下：
		public static String format(String format, Object... args) {
			return new Formatter().format(format, args).toString();
		}
		public static String format(Locale l, String format, Object... args) {
			return new Formatter(l).format(format, args).toString();
		}
3. 如：
		${String.format("%-10.2f%n",125.178)}
4. 具体的格式化规则可以查看DecimalFormat的Java文档。

**5)格式化时间：**
1. 也是通过String.format()方法来进行格式化。
		${d=LocalDate.now().plusDays(2);String.format("%tB %te,%tY%n",d,d,d）}
2. 具体的格式化规则可以查看NumberFormat的Java文档。

---
## 8.在JSP2.0及更高版本配置EL
**1)EL什么情况下需要配置：**
1. EL提倡的是无脚本的JSP,关于JSP脚本的知识可以自行百度。
可以关闭脚本来强制编写无脚本JSP。
2. 当低版本的JSP部署到兼容高版本的容器中时，需要禁用EL表达式计算。
因为低版本的EL不是内置的，所以需要关闭。

**2)配置关闭JSP脚本：**
- 在部署描述文件web.xml中添加如下配置即可：
		<jsp-config>
			<jsp-property-group>
				<url-pattern>*.jsp</url-pattern>
				<scripting-invalid>true</scripting-invalid>
			</jsp-property-group>
		</jsp-config>
- `url-pattern`定义了那些文件需要禁用脚本。
- 一个web.xml中只能配置一项`<jsp-config>`，如果关闭了JSP脚本，又要禁用EL计算，就需要额外配置一个`<jsp-property-group>`。

**3)配置禁用EL计算：**
1. 有两种方式可以禁用EL计算。
禁用后将不会对出现的EL结构进行解析计算。
2. 禁用方式一：
		<%@ page isELIgnored="true" %>
3. 禁用方式二：
在部署描述符web.xml中配置如下：
		<jsp-config>
			<jsp-property-group>
				<url-pattern>*.jsp</url-pattern>
				<el-ignored>true</el-ignored>
			</jsp-property-group>
		</jsp-config>

---
## 9.国际化简介
**1)国际化简介：**
1. 全球化时代需要设计能部署在不同语言国家和地区的应用程序。
2. 需要了解两个术语：
	- 国际化：简称i18n,是开发多语言和数据格式的开发技术，不用重写逻辑。
	- 本地化：简称i10n,将国际化程序改为支持特定语言区域的技术。
3. Java为字符和字符串提供了Unicode支持，所以用Java编写国际化应用时很容易的事情。国际化的方式主要取决于有多少静态数据需要用不同语言来显示。

**2)国际化的两种方式：**
1. 大量数据是静态的，就需要对每一个语言区域单独创建一个版本。
适用于有大量HTML页面的Web应用程序。
2. 如果需要国际化的静态资源有限：
	- 可以将文本元素(如表单信息和错误信息等)隔离为文本文件。
	- 每个文本文件都保存着一个语言区域的所有文本的译文。
	- 应用程序根据所在的语言区域自动获取每一个元素。
3. 将文本元素隔离为文本文件的好处：
文本元素不用再编译应用程序就可以进行编辑。(不用重新部署应用程序)

---
## 10.语言区域
**1)语言区域简介：**
1. Java.util.Locale类表示一个语言区域。
2. Locale类主要有三个原件(构造参数)：
	- language：语言代号
	- country：国家代号
	- variant：特定于供应商或者浏览器的代号(一般不设置)
3. 构造方法如下：
		public Locale(String language, String country, String variant) {
			if (language== null || country == null || variant == null) {
				throw new NullPointerException();
			}
			baseLocale = BaseLocale.getInstance(convertOldISOCodes(language), "", country, variant);
			localeExtensions = getCompatibilityExtensions(language, "", country, variant);
		}
	
		public Locale(String language, String country) {
			this(language, country, "");
		}
	
		public Locale(String language) {
			this(language, "", "");
		}

**2)参数及静态域介绍：**
1. language：语言代号是一个有效的ISO码
ISO 639语言码规范如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-07.jpg)
2. country：国家代号
ISO 3199国家码如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-08.jpg)
3. variant：特定于供应商或者浏览器的代号
	- WIN代表Windows
	- MAC表示macintosh
	- POSIX表示POSIX
	- 等等，多个代号之间用下划线`_`分开
4. Locale类提供了静态域，可以通过静态域来创建Locale对象。
常见静态域：CANADA_FREANCH，CHINA，CHINESE，ENGLISH，FRANCE，FRANCH，US，UK等
通过静态域构造Locale对象：
		Locale locale=Locale.CHINESE;
5. 可以通过计算机的默认语言区域来创建Locale对象：
		Locale locale=Locale.setDefault();

---
## 11.国际化SpringMVC应用程序
国际化SpringMVC应用程序的两个重要步骤：
- 将文本组件隔离成属性文件
- 选择和读取正确的属性文件

**1)将文本组件隔离为属性文件：**
1. 国际化应用将每一个语言区域的文本元素都单独保存在一个独立的属性文件中。
每个文件都包含key-value对，并且每个key都唯一表示某一特定语言的区域对象。
key只能是字符创，value可以是其他任意类型的对象。
2. 如英语，德语和汉语的属性文件示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-09.jpg)
中文的是Unicode码，可以在编辑器中输入中文后转换。
3. `java.util.ResourceBundle`类简介：
可以用于选择和读取特定语言区域的属性文件以及查找值
是一个抽象类，提供了一个getBundle()方法返回子类示例。
具体的使用后面介绍。
4. ResourceBundle有一个基准名称，属性文件名最好包含该基准名称。
5. 属性文件命名需要符合如下规范：
		基准名_语言码_国家码.properties
命名示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-10.jpg)

**2)选择和读取正确的属性文件：**
1. ResourceBundle类的getBundle方法如下：(主要是前两个)
		public static final ResourceBundle getBundle(String baseName){...}
		//baseName就是基准名
		public static final ResourceBundle getBundle(String baseName,Locale locale){...}

		public static final ResourceBundle getBundle(String baseName, Locale targetLocale,Control control) {...}

		public static ResourceBundle getBundle(String baseName, Locale locale,ClassLoader loader){...}

		public static ResourceBundle getBundle(String baseName, Locale targetLocale,ClassLoader loader, Control control){...}
使用示例：
		ResourceBundle rb1=ResourceBundle.getBundle("resources1",Locale.CHINESE);
		ResourceBundle rb2=ResourceBundle.getBundle("resources2");
2. ResourceBundle加载属性文件的过程：
	- 通过getBundle的参数寻找属性文件
		- 找到返回
		- 找不到属性文件则查看有没有默认文件：`baseName.properties`
			- 找到返回
			- 找不到报错`java.util.MissingResourceException`
	- 调用该对象事务getString方法传入一个key获取value值
		- 若未找到key，报错`java.util.MissingResourceException`
3. 简单示例如下：
		//加载属性文件，得到的rb1就是属性文件中的键值对集合
		ResourceBundle rb1=ResourceBundle.getBundle("resources1");
		//获取某一属性的值
		String value=rb1.getString("KEY1");
4. SpringMVC中可以使用message bean来加载属性文件。

**3)使用message bean：**
1. message bean在[SpringMVC学习笔记(四)](https://zjxkenshine.github.io/2018/05/22/SpringMVC%E5%AD%A6%E4%B9%A0%EF%BC%88%E5%9B%9B%EF%BC%89%EF%BC%9A%E8%BD%AC%E6%8D%A2%E5%99%A8%EF%BC%8C%E9%AA%8C%E8%AF%81%E5%99%A8/)的验证器中就已经使用过。
2. 有两个类可以作为message的实现：
	- `ReloadableResourceBundleMessageSource`，推荐使用，
	可以在应用程序的目录下搜索属性文件，修改后会重新加载。
	- `ResourceBundleMessageSource`，只会在/WEB-INF/classes下寻找属性文件，且修改后需要重启JVM才生效。
3. 使用示例：
加载一个属性文件：
		<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
			<property name="basename" value="/WEB-INF/jsp4/messages"></property>
			<property name="defaultEncoding" value="UTF-8"></property>
		</bean>
加载多个属性文件：
		<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
			<property name="basenames">
			<list>
				<value="路径1">
				<value="路径2">
				...
			</list>
			</property>
			<property name="defaultEncoding" value="UTF-8"></property>
		</bean>
注意，id一定得是messageSource，否则会报错。
4. 若配置messageSource时进行了配置了编码：
		<property name="defaultEncoding" value="UTF-8"></property>
则属性文件可以设置为UTF-8格式。

---
## 12.选择区域与message标签
**1)选择应用程序的语言区域：**
1. 用户选择语言区域最简单的方式就是获取浏览器的accept-language标题。
ccept-language标题提供用户偏好那种语言的设置。
2. 在Chrome中的偏好语言可以在设置->高级->语言中设置，将偏好语言移到顶部：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-11.jpg)
3. 也可以通过某个Session属性或者Cookie属性来选择语言区域(书上未介绍)
4. SpringMVC分别为这三种方式提供了区域解析器：
	- AcceptHeaderLocaleResolver（从accept-language获取，推荐）
	- SessionLocaleResolver（从Session获取）
	- CookieLocaleResolver（从Cookie获取）
5. 使用AcceptHeaderLocaleResolver解析器，如果浏览器的`accept-language`能和应用程序支持的某个语言区域匹配，则选择该区域，否则选择默认语言区域。
6. 在配置文件中配置相关的bean就可以了：
		<bean id="localeResolver" 
		class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver">
		</bean>
id也一定得是localeResolver，否则会报错。

**2)使用message标签：**
1. 显示本地化最简单的方式就是使用`<message>`标签。
2. 使用该标签需要在JSP页面开头引入如下uri：
		<%@taglib uri="http://www.springframework.org/tags" prefix="spring" %>
3. 该标签有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-12.jpg)

---
## 13.国际化示例
测试在中文(简体)和英文(美国)下的显示。

**1)控制类及配置代码如下**
1. 控制器类代码：
		@Controller
		public class i18nController {
			@RequestMapping("i18n_form")
			public String i18nForm(){
				return "testForm";
			}
		}
2. springmvc-servlet.xml配置如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans  
		    xmlns="http://www.springframework.org/schema/beans"  
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xmlns:context="http://www.springframework.org/schema/context"
		    xmlns:mvc="http://www.springframework.org/schema/mvc"  
		    xmlns:aop="http://www.springframework.org/schema/aop"   
		    xsi:schemaLocation="http://www.springframework.org/schema/beans   
		                        http://www.springframework.org/schema/beans/spring-beans.xsd  
		                        http://www.springframework.org/schema/aop    
		                        http://www.springframework.org/schema/aop/spring-aop.xsd
		                        http://www.springframework.org/schema/mvc    
		                        http://www.springframework.org/schema/mvc/spring-mvc.xsd
		                        http://www.springframework.org/schema/context       
		                        http://www.springframework.org/schema/context/spring-context.xsd">
			<context:component-scan base-package="Controller5"></context:component-scan>
			<mvc:annotation-driven></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			
			<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
				<property name="basenames">
					<list>
						<value>WEB-INF/jsp5/form</value>
					</list>
				</property>
				<property name="defaultEncoding" value="UTF-8"></property>
			</bean>
			
			<bean id="localeResolver" class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver">
			</bean>
			
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp5/"></property>
		 		<property name="suffix" value=".jsp"></property>
			</bean>
		 </beans>
3. web.xml配置如下：
		...
		<servlet>
			<servlet-name>springmvc</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>/WEB-INF/jsp5/springmvc-servlet.xml</param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>
		...

**2)JSP页面及属性文件：**
1. testForm.jsp内容如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<%@taglib uri="http://www.springframework.org/tags" prefix="spring" %>
		<%@taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
		<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title><spring:message code="testForm.title"/></title>
		</head>
		<body>
		<p>
			locale:${pageContext.response.locale}
		</p>
		<spring:message code="test.message"/>
		
		</body>
		</html>
2. 属性文件form_en_US.properties如下：
		testForm.title=EglishForm
		test.message=this is EglishMessage
默认属性文件和form_zh_CN.properties代码如下：(UTF8编码)
		testForm.title=中文表单
		test.message=这是中文的信息

**3)测试：**
1. 中文环境下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-13.jpg)
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-14.jpg)
2. 英文环境下：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-15.jpg)
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00068SpringMVC%E5%AD%A6%E4%B9%A06-16.jpg)
3. 未重新部署项目而能看到不同的语言显示，这就是国际化和本地化。

---