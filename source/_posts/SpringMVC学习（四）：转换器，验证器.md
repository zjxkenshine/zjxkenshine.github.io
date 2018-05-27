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
	- 验证器简介
	- Spring验证器
	- JSR303验证器

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
## 4.验证器Validater简介
1. Converter和Formatter作用于field(字段)级别。
验证器则是Object级别的，决定某一个对象中的所有field是否有效以及是否遵循某些规则。
2. 程序中既有Converter/Formatter又有验证器时的调用顺序：
**在调用控制器Controller期间，如果有字符串需要转换为指定的类型则会调用相应的Formatter，格式化成功后校验器(验证器)介入判断所有field是否遵循规则。**
3. 例如生日字段，将String转换为Date类型后需要用校验器来判断该时间是否大于当前日期。
虽然可以将该判断写在转换器中，但是使用验证器更加合适。
验证器可以检验两个或者更多字段之间的关系或者它们本身需要满足的条件。
4. 简单的示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-07.jpg)
5. Spring自带了验证器，不过更加推荐使用JSR 303验证器。
(JSR 303是一种Java验证规范)

---
## 5.Spring验证器简单使用
**1)Spring验证器的简单使用：**
1. 使用Spring验证器需要实现org.springframework.validation.Validator接口，该接口代码如下：
		public interface Validator {
			boolean supports(Class<?> clazz);
			void validate(Object target, Errors errors);
		}
2. supports表示是否可以处理指定的Class,可以处理则返回true并使用validate验证目标对象，验证错误填入Error对象。
3. Errors是一个在org.springframework.validation包下的接口。
Errors对象中包含了一系列FileError和ObjectError对象，分别代表字段出错和对象出错，但是单独创建者两个对象非常麻烦，有很多构造参数：
		public FieldError(
				String objectName, String field, Object rejectedValue, boolean bindingFailure,
				String[] codes, Object[] arguments, String defaultMessage) {
		}
		public ObjectError(String objectName, String[] codes, Object[] arguments, String defaultMessage) {
		}
所以一般使用Errors接口的reject方法(ObjectErrors)和rejectValue方法(FieldErrors)。
4. reject方法和rejectValue方法声明如下：
		void reject(String errorCode);
		void reject(String errorCode, String defaultMessage);
		void reject(String errorCode, Object[] errorArgs, String defaultMessage);
		
		void rejectValue(String field, String errorCode);
		void rejectValue(String field, String errorCode, String defaultMessage);
		void rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage);
通常只给reject或者rejectValue传入一个错误码(errorCode),然后会在外部定义的文件去找(详见第三点错误信息源文件)，也可以添加一个默认错误消息defaultMessage，错误消息文件中没有该错误码时使用默认消息。
5. 使用示例：
在validate重写方法内使用：
		errors.reject("code");	//ObjectErrors,对象级别错误
		errors.rejectValue("price","code");	//字段，错误码，FieldError

**2)ValidationUtils类：**
1. 可以简化errors.rejectValue()的验证。
类：org.springframework.validation.ValidationUtils
2. 两个重要方法：
		ValidationUtils.rejectIfEmpty(errors, "name","nameNotNull");
相当于：
		if(name==null||name.isEmpty()){	//不为空
			errors.rejectValue("name","nameNotNull");
		}
方法二：
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name","nameNotNull");
相当于：
		if(name==null||name.trim().isEmpty()){	//去掉空格后不为空
			errors.rejectValue("name","nameNotNull");
		}
3. ValidationUtils类的这两个方法重构方法如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-08.jpg)
4. ValidationUtils还有一个方法：
		public static void invokeValidator(Validator validator, Object obj, Errors errors) {
			invokeValidator(validator, obj, errors, (Object[]) null);
		}
	
		public static void invokeValidator(Validator validator, Object obj, Errors errors, Object... validationHints) {
		...
		}
可以用来调用验证器。

**3)定义错误消息源文件：**
1. 如果要从某一属性文件获取错误消息，需要在配置文件springmvc.xml中添加messageSource bean的声明:
		...
		<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
			<property name="basename" value="/WEB-INF/resource/message"></property>
			<property name="defaultEncoding" value="UTF-8"></property>
		</bean>
2. 然后再对应的/WEB-INF/resource目录下创建message.properties文件
文件里面的内容就是：
		错误码=错误信息
		nameNotNull=name can not be null
这种格式，定义前端显示的提示信息。
3. 然后再在Controller中显式调用Validator。(详见例子)
4. 注意需要设置defaultEncoding属性，否则会乱码。

**4)全局验证器**
1. 一般情况下不需要显式注册Validator。
默认请款下的验证器只能在当前控制器中使用。
2. 如果需要实现全局校验，需要配置Springmvc.xml文件：
		...
		<mvc:annotation-driven validator="userValidator"/>
		<bean id="userValidator" class="com.xxx.xxx.xxxValidator"/>
		...
3. Validator的使用(包括转配bean)都和一般类没啥区别。

---
## 6.Spring验证器测试
使用上面Formatter的测试代码进行修改。

**1)添加EmployeeValidator验证器：**
1. 验证要求，各字段不能为空，生日要小于当前时间。
2. 在Controller4包下创建EmployeeValidator验证器，代码如下：
		public class EmployeeValidator implements Validator{
			@Override
			public boolean supports(Class<?> clazz) {
				return Employee.class.isAssignableFrom(clazz);
			}
		
			@Override
			public void validate(Object target, Errors errors) {
				Employee employee=(Employee)target;
				ValidationUtils.rejectIfEmptyOrWhitespace(errors,"firstName","firstName.notnull","firstname不能为空");
				ValidationUtils.rejectIfEmptyOrWhitespace(errors,"lastName","lastName.notnull","lastName不能为空");
				ValidationUtils.rejectIfEmptyOrWhitespace(errors,"birthDate","birthDate.notnull","生日不能为空");
				Date birthday=employee.getBirthDate();
				if(birthday!=null){
					if(birthday.after(new Date())){
						System.out.println("生日日期晚于当前日期");
						errors.rejectValue("birthDate","birthDate.error","生日不能晚于当前时间");
					}
				}
			}
		}
3. 仅仅做Employee的校验，所以不用在springmvc-servlet.xml配置文件中声明该验证器的bean。
4. 在这里配置了默认消息后，可以省略配置外部消息源文件这一步。

**2)配置错误消息源文件：**
1. 在springmvc-servlet.xml声明messageSource bean：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans ...>
			<context:component-scan base-package="Controller4"></context:component-scan>
			<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			
			<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
				<property name="formatterRegistrars">
					<set>
						<bean class="Controller4.S2DFormatterRegister" name="formatregister1">
							<constructor-arg type="java.lang.String" value="yyyy-MM-dd"></constructor-arg>
						</bean>
					</set>
				</property>
			</bean>
			<!--配置错误消息源-->
			<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
				<property name="basename" value="/WEB-INF/jsp4/messages.properties"></property>
				<property name="defaultEncoding" value="UTF-8"></property>
			</bean>
			
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp4/"></property>
		 		<property name="suffix" value=".jsp"></property>
		 	</bean>
		 </beans>
2. 在对应的目录/WEB-INF/jsp4/下创建messages.properties,内容如下：
		firstName.notnull=firstname not null
		lastName.notnull=lastname不能为空
		birthDate.notnull=生日不能为空
		birthDate.error=生日不能晚于当前时间
validator中的错误码就到这个文件中来查找。(未配置则使用默认消息提示)

**3)控制器：**
1. EmployeeController代码修改如下：
		@Controller
		public class EmployeeController {
			private static final Log logger=LogFactory.getLog(EmployeeController.class);
			//表单初始化
			@RequestMapping(value="employee_input")
			public String inputEmployee(Model model){
				logger.info("调用inputEmployee初始化方法");
				model.addAttribute("employee",new Employee());
				return "AddEmployeeForm";
			}
			//表单处理
			@RequestMapping(value="employee_save")
			public String saveEmployee(@ModelAttribute Employee employee,BindingResult bindingResult,Model model){
				logger.info("调用saveEmployee请求处理方法");
				
				EmployeeValidator employeeValidator=new EmployeeValidator();	//显式调用验证器
				employeeValidator.validate(employee, bindingResult);
				
				if(bindingResult.hasErrors()){
					FieldError fieldError=bindingResult.getFieldError();
					logger.info("出错了:Code="+fieldError.getCode()+",Field="+fieldError.getField());
					return "AddEmployeeForm";
				}
				return "AddSuccess";
			}
		}
2. 并将条件页面的errors表单修改为
		<form:errors path="*"/><br>
其余代码不变。
3. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-09.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-10.jpg)
4. 如果引入外部源文件一定要在配置文件中添加：
		<property name="defaultEncoding" value="UTF-8"></property>
否则会乱码：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-11.jpg)

**4)使用EmployeeValidator的两种方式：**
1. 方式一：推荐
在控制器中显式使用：
		...
		EmployeeValidator employeeValidator=new EmployeeValidator();
		employeeValidator.validate(employee, bindingResult);
		...
2. 方式二：
在控制器中编写initBinder方法，并将验证器传到WevDataBinder，并调用其validate方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-12.jpg)

---
## 7.JSR303验证器
**1)JSR303简介：**
1. JSR 303不是一种具体的技术，而是一种验证规范文档。于2009年发布。
同样的还有JSR 349(2013年发布)。
2. JSR 303是正式的Java规范，所以最好使用JSR 303而不是Spring验证器。
3. JSR Bean Validation有两个实现：
	- Hibernate Validator（推荐使用）
	- Apache Bval
4. JSR 349暂时不做讨论。

**2)JSR303约束文件**
1. 下载地址：
<https://jcp.org/en/jsr/detail?id=303>
2. JSR303不需要编写验证器，但是需要根据JSR标注类型嵌入约束。
详见下载的文件，这里只是简单介绍。
3. JSR303约束如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-13.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-14.jpg)
4. Hibernate validator 在JSR303的基础上对校验注解进行了扩展，扩展注解如下：
		 @Email			被注释的元素必须是电子邮箱地址
		
		 @Length		被注释的字符串的大小必须在指定的范围内
		
		 @NotEmpty		被注释的字符串的必须非空
		
		 @Range			被注释的元素必须在合适的范围内
5. 关于非空：
		@NotEmpty 用在集合类上面
		@NotBlank 用在String上面
		@NotNull 用在其他类型上
清测@NotNull对String无效。

**3)如何设置错误消息：**
1. 和Spring验证器一样，也需要自定义外部属性文件message.properties(自己指定的)来覆盖来自JSR303的默认消息。
2. 其中错误码的格式为：
`constranint.object.property`
constranint：注解名称
object：对象
property：属性
3. 如使用@Past标注Product对象的addDate属性,则对应的错误码如下：
`Past.product.addDate`
中间的对象名取决于@Valid注解标注的哪个对象。

---
## 8.JSR303验证器使用测试
使用起来比spring自带的验证器简单多了。

**1)使用流程**
1. 添加依赖如下：
		...
		<dependency>
		  <groupId>org.hibernate.validator</groupId>
		  <artifactId>hibernate-validator</artifactId>
		  <version>6.0.9.Final</version>
		</dependency>
		<dependency>
		  <groupId>javax.validation</groupId>
		  <artifactId>validation-api</artifactId>
		  <version>2.0.1.Final</version>
		</dependency>
		...
2. 使用JSR303注解标注Employee类：
		public class Employee implements Serializable{
			private static final long serialVersionUID = 186417813894448409L;
			private long id;
			@NotBlank
			private String firstName;
			@NotBlank
			private String lastName;
			@NotNull
			@Past
			private Date birthDate;
			private int salaryLevel;
			//get,set方法
		}
3. 修改EmployeeController代码如下：
		@Controller
		public class EmployeeController {
			private static final Log logger=LogFactory.getLog(EmployeeController.class);
			//表单初始化
			@RequestMapping(value="employee_input")
			public String inputEmployee(Model model){
				logger.info("调用inputEmployee初始化方法");
				model.addAttribute("employee",new Employee());
				return "AddEmployeeForm";
			}
			//表单处理(注意在Employee前添加了@Valid)
			@RequestMapping(value="employee_save")
			public String saveEmployee(@Valid @ModelAttribute Employee employee,BindingResult bindingResult,Model model){
				logger.info("调用saveEmployee请求处理方法");
				if(bindingResult.hasErrors()){
					FieldError fieldError=bindingResult.getFieldError();
					logger.info("出错了:Code="+fieldError.getCode()+",Field="+fieldError.getField());
					return "AddEmployeeForm";
				}
				return "AddSuccess";
			}
		}
4. 注册messageSource Bean：(springmvc-servlet.xml中)
		<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
			<property name="basename" value="/WEB-INF/jsp4/messages"></property>
			<property name="defaultEncoding" value="UTF-8"></property>
		</bean>
5. 在/WEB-INF/jsp4/messages.properties文件中添加相关注册码：
		NotBlank.employee.firstName=jsr303:firstname不能为空
		NotBlank.employee.lastName=jsr303:lastName不能为空
		NotNull.employee.birthDate=jsr303:birthDate不能为空
		Past.employee.birthDate=jsr303:birthDate不能晚于当前时间

**2)测试：**
1. 为空：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-15.jpg)
2. 格式化Formatter错误：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-16.jpg)
3. 日期超过今天：
![](http://p5ki4lhmo.bkt.clouddn.com/00066SpringMVC%E5%AD%A6%E4%B9%A04-17.jpg)
4. 简单的JSR303的用法就是这些。

---