---
title: SpringMVC学习（四）：转换器，验证器
date: 2018-5-22 22:17:10 
tags: 
- Spring
- Spring MVC
categories: Java框架

---
## 0.学习准备
1. 参考资料：
参考书籍《Spring MVC学习指南》(第二版)(基础学习)
《Spring实战》(第四版)(综合运用)
参考视频：
尚硅谷《Spring MVC》(补充知识)
2. 简单目录：
	- 转换器Converter
	- 格式化Formatter
	- 用Registrar注册Formatter
	- Spring内置的各种转换器

---
## 1.转换器与格式化简介
1. Sring的数据绑定可以进行简单的数据绑定。
但是在涉及日期的数据绑定Spring还是会显得杂乱无章。
2. Spring总是试图使用默认语言区域的日期格式将日期输入绑定到java.util.Date
如美国的默认日期格式是`月/日/年`，如果输入的字符串是"2018-5-23",那么绑定到java.util.Date就会出现错误。
3. 为了解决数据绑定时的数据转换(绑定不同的日期样式)，就需要使用转换器(Converter)或者格式化(Formatter)来协助完成。
4. Converter与Formatter的区别：
	- Converter是通用原件，可以在应用程序的任意层总使用。且转换与被转换的数据类型没有限制。
	- Fomatter是转么为Web层设计的，只能从String类型转换为其他数据类型。

---
## 2.转换器Convert
**Convert简介：**
1. Spring的Convert可以将一种类型转换成另一种类型的一个对象。
2. 创建的Converter需要实现一个org.springframwork.core.convert.converter.Converter接口。
该接口声明如下：
		public interface Converter<S,T>
S表示源类型(被转换类型)
T表示目标类型(要转换成的类型)
3. 需要在实现的Converter中实现一个方法：
		S conver(T tsource)
4. 如String到Date的转换器可以这样声明：
		public class MyConverter implements Converter<String,Date>{
			@override
			public Date convert(String s){...}
		}
5. Long到Date的转换器可以这样声明：
		public class MyConverter implements Converter<Long,Date>{
			@override
			public Date convert(String s){...}
		}

###	Convert使用示例
1. 简单目录如下：(XXX4)
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-01.jpg)
2. 只是一个简单的添加表单，添加成功则跳转到成功页面，日期格式化失败则返回显示。
只是为了测试关于日期类型的转换器Controller。

**1)转换器与模型：**
1. 一个适用于任何日期格式的String2Date转换器：(Controller4包中)
		public class S2DConverter implements Converter<String, Date>{
			private String datePattern;
			
			public S2DConverter(String dateParttern) {
				this.datePattern=dateParttern;
			}
		
			@Override
			public Date convert(String source) {
				try {
					//日期格式化类
					SimpleDateFormat dateFormat=new SimpleDateFormat(datePattern);
					//禁止自动计算，否则20个月会被转换为1年8个月
					dateFormat.setLenient(false);
					return dateFormat.parse(source);
				} catch (ParseException e) {
					//向上抛出异常,会被<form:errors>表单打印
					throw new IllegalArgumentException("格式不正确");	
				}
			}
		}
2. 要使用Conveter必须要在SpringMVC配置一个org.springframwork.context.support.ConversionServiceFactoryBean类的Bean。这个Bean需要包含一个converters属性，该属性值是应用程序中用到的所有自定义Converter列表。
配置类代码如下：
		<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
			<property name="converters">
				<list>
					<bean class="Controller4.S2DConverter"  name="convert1">
						<constructor-arg type="java.lang.String" value="yyyy-MM-dd"></constructor-arg>
					</bean>
				</list>
			</property>
		</bean>
3. 最后要在`<mvc:annotion-driver>`元素的conversion-service属性配置上该conversionService Bean的ID：(这里的id是conversionService)
		<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
4. 关于`<mvc:annotion-driver>`的功能:
`<mvc:annotation-driven>`会自动注册RequestMappingHandlerMapping与RequestMappingHandlerAdapter两个Bean,这是Spring MVC为@Controller分发请求所必需的，并且提供了数据绑定支持，@NumberFormatannotation支持，@DateTimeFormat支持,@Valid支持读写XML的支持（JAXB）和读写JSON的支持（默认Jackson）等功能。
5. 模型实体类Model4.Employee代码如下：
		public class Employee implements Serializable{
			private static final long serialVersionUID = 186417813894448409L;
			private long id;
			private String firstName;
			private String lastName;
			private Date birthDate;
			private int salaryLevel;
			//getset和toString方法
		}

**2)控制器类：**
1. Controller4.EmployeeController代码如下：
		@Controller
		public class EmployeeController {
			private static final Log logger=LogFactory.getLog(EmployeeController.class);
			//表单初始化
			@RequestMapping(value="employee_input")
			public String inputEmployee(Model model){
				model.addAttribute("employee",new Employee());
				return "AddEmployeeForm";
			}
			//表单处理
			@RequestMapping(value="employee_save")
			public String saveEmployee(@ModelAttribute Employee employee,BindingResult bindingResult,Model model){
				logger.info("调用saveEmployee请求处理方法");
				if(bindingResult.hasErrors()){
					FieldError fieldError=bindingResult.getFieldError();
					logger.info("出错了:Code="+fieldError.getCode()+",Field="+fieldError.getField());
					return "AddEmployeeForm";
				}
				return "AddSuccess";
			}
		}
2. 表单处理方法中的BindingResult类型参数放置了Spring的所有数据绑定错误。
该方法利用BindingResult记录所有绑定错误，可以利用`<form:errors>`标签将其显示在一个表单中。

**3)视图类：**
1. WEB-INF/jsp4/AddEmployeeForm.jsp代码如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>添加员工</title>
		</head>
		<body>
		<form:form commandName="employee" action="employee_save" method="post">
			<form:errors path="birthDate"/><br>
			firstname：<form:input path="firstName" tabindex="1"/><br>
			lastname：<form:input path="lastName" tabindex="2"/><br>
			生日：<form:input path="birthDate" tabindex="3"/><br>
			<input type="reset" tabindex="4"><input type="submit" tabindex="5">
		</form:form>
		</body>
		</html>
2. 添加成功页面：(AddSuccess.jsp)
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>添加成功</title>
		</head>
		<body>
		添加成功
		</body>
		</html>

**4)配置：**
1. web.xml配置：
		...
		<servlet>
			<servlet-name>springmvc</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>/WEB-INF/jsp4/springmvc-servlet.xml</param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>
		...
其他配置和前面的配置文件相同。
2. WEB-INF/jsp4/springmvc-servlet.xml最终配置如下：
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
			<context:component-scan base-package="Controller4"></context:component-scan>
			<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			
			<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
				<property name="converters">
					<list>
						<bean class="Controller4.S2DConverter" name="convert1">
							<constructor-arg type="java.lang.String" value="yyyy-MM-dd"></constructor-arg>
						</bean>
					</list>
				</property>
			</bean>
			
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp4/"></property>
		 		<property name="suffix" value=".jsp"></property>
		 	</bean>
		 </beans>

**5)测试：**
1. 日期格式错误添加失败：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-02.jpg)
添加失败日志：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-03.jpg)
2. 添加成功：(日期填2018-5-23)
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-04.jpg)

---
## 3.格式化Formatter
**1)Formatter简介：**
1. Formatter也是将一种类型转换为另一种类型。
Formatter的源类型必须是一个String，而Converter可以是任何类型
Formatter更适合于web层，而Converter可以在其他层使用。
2. 如果仅仅是为了转换SpringMVC的表单输入，推荐使用Formatter

**2)简单使用Formatter(一般用法)：**
1. 创建Formatter需要实现org.pringframework.format.Formatter接口。
该接口声明如下：
		public interface Formatter<T>
2. 需要实现两个接口方法：
		T parse(String text,java.util.Locale locale)
		String print(T object,java.util.Locale locale)
parse方法利用指定的locale将一个String解析成目标类型。
print方法相反，返回目标对象的字符串表示。
3. 是需要在配置xml文件中添加org.springframwork.context.support.FormattingConversionServiceFactoryBean类型的Bean,包含了一个formatters属性，该属性包含了formatter bean集合。（类似Converter的列表，通过转换服务来装配formatter）

### 一般使用方式
1. 使用Converter的测试代码进行修改
2. 格式化类Controller4.S2DFormatter代码如下：
		public class S2DFormatter implements Formatter<Date>{
			
			private SimpleDateFormat simpleDateFormat;
			
			public S2DFormatter(String datePattern) {
				simpleDateFormat=new SimpleDateFormat(datePattern);
				simpleDateFormat.setLenient(false);
			}
			
			@Override
			public String print(Date object, Locale locale) {
				return simpleDateFormat.format(object);
			}
			
			@Override
			public Date parse(String text, Locale locale){
				try {
					return simpleDateFormat.parse(text);
				} catch (ParseException e) {
					throw new IllegalArgumentException("格式错误");
				}
			}
		}
3. springmvc-servlet.xml代码修改如下：(将注册的Converter换成Formatter)
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
			<context:component-scan base-package="Controller4"></context:component-scan>
			<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			<!--注册装配Formatter-->
			<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
				<property name="formatters">
					<set>
						<bean class="Controller4.S2DFormatter" name="formater1">
							<constructor-arg type="java.lang.String" value="yyyy-MM-dd"></constructor-arg>
						</bean>
					</set>
				</property>
			</bean>
			
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp4/"></property>
		 		<property name="suffix" value=".jsp"></property>
		 	</bean>
		 </beans>
4. 其余代码不变，测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-05.jpg)

### 通过Register注册Formatter
1. Formatter的另一种使用方式。
使用FormatterRegistrar接口的实现类来注册Formmatter。
2. 在以上代码的基础上，在Controller4添加一个Register，实现FormatterRegistrar接口,代码如下：
		public class S2DFormatterRegister implements FormatterRegistrar{
			private String dateParttern;
			public S2DFormatterRegister(String dateParttern) {
				this.dateParttern=dateParttern;
			}
		
			@Override
			public void registerFormatters(FormatterRegistry registry) {
				registry.addFormatter(new S2DFormatter(dateParttern));
			}
		}
3. 修改springmvc-servlet.xml，将装配Formatter列表改为装配FormatterRegistrar列表：
		......
		<!--注册装配FormatterRegister-->
		<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
			<property name="formatterRegistrars">
				<set>
					<bean class="Controller4.S2DFormatterRegister" name="formatregister1">
						<constructor-arg type="java.lang.String" value="yyyy-MM-dd"></constructor-arg>
					</bean>
				</set>
			</property>
		</bean>
		......
4. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-06.jpg)

---