---
title: SpringMVC学习（一）：MVC模式与Spring MVC简单使用
date: 2018-5-3 16:42:39  
tags: 
- Spring
- Spring MVC
categories: Java框架

---
## 0.学习准备
1. 参考资料：
参考书籍《Spring MVC学习指南》(第二版)(基础学习)
《Spring实战》(第四版)(综合运用)

2. 学习目录：
	- HTTP协议简介
	- Java Web设计模型
	- MVC模式
	- 简单的Spring MVC应用构建

---
## 1.HTTP简介
**1)相关知识：**
1. HTTP：超文本传输协议
HTTP使得Web服务器与浏览器之间可以通过互联网或者内网进行数据交互。由万维网联盟(W3C)负责维护HTTP。
Web服务器24小时运行并等待HTTP客户端(通常是Web浏览器)来连接并请求资源。一般是客户端主动发起一个连接来连接客户端。
目前最新版本是HTTP/2，于2015年发布。
2. 2011年标准化组织IETE发布了WebSocket协议，该协议可以使一个HTTP连接升级为WebSocket连接，支持双向通信，使得Web服务器可以通过WebSocket协议主动发起和客户端的通信。
3. 互联网用户需要通过点击或者输入一个URL连接或者地址来访问一个资源。
4. 各版本HTTP发布时间：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-01.jpg)

**2)URL介绍：**
1. HTTP的URL格式如下：
		protocl://[host.]domain[:port][/context][/resource][?query|path viriable]
或者：
		protocl://IP[:port][/context][/resource][?query|path viriable]
2. protocl：协议
代表所采用的协议，一般是http/https，也可以是其他的。
3. host：主机名
用来表示在互联网或者内网中的唯一地址。
最常见的是www.,所以作为了默认的主机名。如访问`http://baidu.com`映射的就是`http://www.baidu.com`,也有其他的。
4. IP：ip地址
用于直接访问该ip的资源，可以用ping命令来获取对应域名的IP地址。
5. domain：域名
ip不方便，所以使用域名访问。一台计算机可以托管多个域名。所以不同的域名可能指向同一个ip。
6. port：端口号，默认为80端口，如果web服务器运行在端口上则无需使用端口号。
但是Tomcat的默认端口是8080，所以访问时需要使用：`localhost:8080...`
7. context：上下文，代表应用名称
可选，一台Web服务器可以运行多个上下文(应用)，其中一个可以配置为默认上下文。若访问这个默认的上下文则可以跳过context部分。
8. resource：资源
一个context(应用)可以有一个或者多个默认资源(default.html或者index.html等)，存在多个默认资源且未指定时会返回最高优先级的。
9. ?query|path viriable：查询语句与路径参数
查询语句是key/value组，多个则由&分开。
路径参数类似于查询语句，只有value部分，用/分开。

**3)HTTP请求：**
1. HTTP请求示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-02.jpg)
2. HTTP请求包含三部分内容：
	- 方法-URI-协议版本
	- 请求头信息
	- 请求正文(内容)
3. 第一行代表的就是“方法-URI-协议版本”
POST是方法，/example/default.jsp是URI，协议版本是HTTP 1.1。
	- HTTP 1.1定义了7种类型的方法：GET,POST,HEAD,OPTIONS,PUT,DELETE以及TRACE。前两个广泛应用。
	- URI定义了一个互联网资源，通常是服务器根目录的相对路径。通常以`/`开头。URL是URI的一个具体类型。
4. 请求头信息包含关于客户端环境以及视图内容等有用信息。
每项头信息都使用回车/换行分隔。(CRLF)
5. 请求正文，所请求的内容，用一个空行与请求头信息区分。

**4)HTTP响应：**
1. HTTP响应示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-03.jpg)
2. HTTP响应包含三部分内容：
	- 协议-状态码-描述
	- 响应头信息
	- 响应正文
3. 第一行代表的就是“协议-状态码-描述”：
200状态码代表成功,401未授权，405使用被禁用的请求方法，404找不到资源。
4. 响应头信息与请求头信息一样也包含了大量消息。
5. 响应正文是一个HTML文档。

---
## 2.Servlet与JSP
1. Servlet技术是Java技术体系中开发Web应用的底层技术。
一开始用来代替CGI技术，随后标准化来支持产生Web动态内容。
2. JSP与1999年为了简化Servlet开发而发布。
3. 一个Servlet是一个Java程序，一个Servlet应用包含了一个或者多个Servlet,一个JSP页面会被翻译并编译成一个Servlet。
4. 典型的Servlet/JSP应用架构：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-04.jpg)
5. 一个Servlet/JSP容器是一个能处理Servlet以及静态资源的Web服务端。
常见的Servlet/JSP容器有Tomcat和Jetty。
6. J2EE(Java企业级应用)，包括了：JMS,EJB,JSP,Servlet和JPA等。

---
## 3.MVC模式基本模型
1. 第一种模型：(只适用与小应用)
通过链接的方式进行JSP页面间的跳转。
2. 第二种模型：基于模型-视图-控制器的MVC模式。
一个MVC模式的应用包含模型、视图、控制器3个模块。
	- 模型：封装了应用的数据和业务逻辑。(JavaBean/POJO)
	- 视图：负责应用的展示。(JSP)
	- 控制器：接收用户输入，改变模型以及调整视图的显示。（Servlet或Filter）
3. 大多数Web框架都是MVC模型的实现。(Spring,Strust,Strust2)
4. MVC模式的架构图：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-05.jpg)

---
## 4.SpringMVC简介
**1)Spring mvc简介：**
1. 开发一个MVC模式的应用，需要编写一个Dispatcher servlet和控制类。
2. Dispatcher servlet必须能够做如下事情：
	- 根据URI调用相对应的action。
	- 实例化正确的控制类。
	- 根据请求参数来构造表单bean。
	- 调用控制器对象的相应方法。
	- 重定向到一个视图（JSP页面）
3. Spring MVC是一个包含了Dispatcher servlet的MVC框架。它调用控制器方法并转发到视图。**不需要编写Dispatcher servlet**。
4. Spring MVC的好处：
	- Spring MVC中提供了一个Dispatch Servlet,无需额外的开发。
	- Spring MVC中时基于XML的配置文件，可以编辑，无需重新编译应用。
	- Spring MVC实例化控制器，并根据用户输入来构造bean。
	- Spring MVC可以自动绑定用户输入，并正确转换数据类型。例如：Spring MVC能自动解析字符串，并设置成float或者decimal类型的属性。
	- Spring MVC可以校验用户的输入，若校验不通过，则重定向回输入表单。输入校验是可选的，支持编程方式以及声明。(内置了常见的校验器)
	- Spring MVC是Spring框架的一部分。可以利用Spring提供的其他功能。
	- Spring MVC支持国际化和本地化。支持根据用户区域显示多国语言。
	- 支持多种视图技术，最常见的JSP包括其他的一些技术（FreeMaker等）。

**2)Spring MVC的DispatcherServlet：**
1. SpringMVC自带了一个开箱即用的DispatcherServlet：
`org.springframework.web.servlet.DispatcherServlet`
2. 需要在web.xml中配置如下来声明使用：
		<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
		<servlet>
			<servlet-name>springmvc</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>/WEB-INF/config/simple-config.xml</param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>
	
		<!-- Map all requests to the DispatcherServlet for handling -->
		<servlet-mapping>
			<servlet-name>springmvc</servlet-name>
			<url-pattern>/</url-pattern>
		</servlet-mapping>
3. `<init-param>`标签：需要初始化的参数
`<param-name>`标签的contextConfigLocation是配置文件路径参数名。
`<param-value>`标签配置的是配置文件的路径。
4. `<load-on-startup>`标签：是否在启动的时候实例化并调用其init()方法的优先级。
值可以认为是它的启动顺序(值越小优先级越高)。
小于或者等于0时只有在第一次请求时才会加载servlet。
5. `<servlet-name>`配置的是servlet名字，这里配的是springmvc，在DispatcherServlet的初始化过程中，框架会在web应用的配置文件路径(一般是/WEB-INF)下寻找名为`springmvc-servlet.xml`的配置文件，生成文件中定义的bean。
而上面的contextConfigLocation参数值则可以配置这个配置文件路径。
6. url-parttern是一个匹配规则：
当servlet容器接收到浏览器发起的一个url请求后，容器会用url减去当前应用的上下文路径，以剩余的字符串作为servlet映射。
假如url是`http://localhost:8080/appDemo/index.html`，其应用上下文是appDemo，容器会将`http://localhost:8080/appDemo`去掉。
可以配置多个url-parttern。
详细参考：
[servlet的url-pattern匹配规则](https://www.cnblogs.com/canger/p/6084846.html)

**3)Controller接口：**
1. 有两种方式实现控制器：
	- 基于org.springframework.web.servlet.mvc.Controller接口实现(spring2.5之前唯一的方式)
	- 基于注解的实现。
2. 该接口公开了一个handleReqiest方法：
		ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
其实现类可以访问对应与请求的HttpServlet和HttpServletResponse，还必须返回一个包含视图路径或视图路径和模的ModelAndView对象。
3. Controller接口的实现类智能处理一个单一对象(Action)，而基于@Controller注解的控制器可以同时支持多个请求处理动作而无需实现任何接口。

---
## 5.MVC应用的简单构建
实现效果可以先看最后。
**1)部署描述符文件和配置文件：**
1. 部署描述文件(web.xml)：
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
		  <display-name>springmvc</display-name>
		  <welcome-file-list>
		    <welcome-file>index.html</welcome-file>
		    <welcome-file>index.htm</welcome-file>
		    <welcome-file>index.jsp</welcome-file>
		    <welcome-file>default.html</welcome-file>
		    <welcome-file>default.htm</welcome-file>
		    <welcome-file>default.jsp</welcome-file>
		  </welcome-file-list>
			<servlet>
				<servlet-name>springmvc</servlet-name>
				<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
				<load-on-startup>1</load-on-startup>
			</servlet>
			<servlet-mapping>
				<servlet-name>springmvc</servlet-name>
				<url-pattern>/</url-pattern>
			</servlet-mapping>
		</web-app>
`<url-pattern>`标签配置为`/`则将所有的URL都映射到该servlet。
未配置init-param元素，Spring MVC的配置文件在/WEB-INF文件夹下。
2. SpringMVC配置文件：
在WEB-INF下创建SpringMVC的配置文件springmvc-servlet.xml，配置如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans
			xmlns="http://www.springframework.org/schema/beans"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:context="http://www.springframework.org/schema/context"
			xmlns:aop="http://www.springframework.org/schema/aop"
			xsi:schemaLocation="http://www.springframework.org/schema/beans
		                        http://www.springframework.org/schema/beans/spring-beans.xsd  
		                        http://www.springframework.org/schema/aop    
		                        http://www.springframework.org/schema/aop/spring-aop.xsd  
		                        http://www.springframework.org/schema/context       
		                        http://www.springframework.org/schema/context/spring-context.xsd">
			<bean name="/product_input.action" class="Controller.InputProductController">
			</bean>
			<bean name="/product_save.action" class="Controller.SaveProductController">
			</bean>
		</beans>

**2)实体类：**
1. Product类：
		public class Product implements Serializable{
			private static final long serialVersionUID = -2111050538757882292L;
			private String name;
			private String discription;
			private float price;
			public String getName() {
				return name;
			}
			public void setName(String name) {
				this.name = name;
			}
			public String getDiscription() {
				return discription;
			}
			public void setDiscription(String discription) {
				this.discription = discription;
			}
			public float getPrice() {
				return price;
			}
			public void setPrice(float price) {
				this.price = price;
			}
		}
2. ProductForm类：
		public class ProductForm {
			private String name;
			private String discription;
			private String price;
			public String getName() {
				return name;
			}
			public void setName(String name) {
				this.name = name;
			}
			public String getDiscription() {
				return discription;
			}
			public void setDiscription(String discription) {
				this.discription = discription;
			}
			public String getPrice() {
				return price;
			}
			public void setPrice(String price) {
				this.price = price;
			}
		}

**3)View视图：**
1. 产品表单jsp页面如下：(/WEB/INF/ProductForm.jsp)
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>Product Form</title>
		</head>
		<body>
		<form action="product_save.action" method="post">
			<fieldset>
				<legend>添加一个产品</legend>
				<label for="name">产品名：</label>、
				<input type="text" id="name" name="name" value="" tabindex="1">
				<label for="description">描述：</label>
				<input type="text" id="description" name="description" tabindex="2">
				<label for="price">价格:</label>
				<input type="text" id="price" name="price" tabindex="3">
				<input type="reset" id="reset" tabindex="4">
				<input type="submit" id="submit" tabindex="5">
			</fieldset>
		</form>
		</body>
		</html>
2. 产品详情的表单如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>商品详情</title>
		</head>
		<body>
		<p>
			商品名称: ${product.name}<br/>
			描述: ${product.discription}<br>
			价格：${product.price}
		</p>
		</body>
		</html>

**4)控制器类：(实现接口方式)**
1. InputProductController类：
		public class InputProductController implements Controller {
			private static final Log logger = LogFactory.getLog(InputProductController.class);
			@Override
			public ModelAndView handleRequest(HttpServletRequest request,
					HttpServletResponse response) throws Exception {
				logger.info("调用InputProductController");
				return new ModelAndView("/WEB-INF/isp/ProductForm.jsp");
			}
		}
2. SaveProductController类：
		public class SaveProductController implements Controller {
			private static final Log logger = LogFactory.getLog(SaveProductController.class);
			@Override
			public ModelAndView handleRequest(HttpServletRequest request,
					HttpServletResponse response) throws Exception {
				logger.info("调用SaveProductController");
				//注意需要转换编码
				request.setCharacterEncoding("UTF-8");
				ProductForm profForm=new ProductForm();
				profForm.setName(request.getParameter("name"));
				profForm.setDiscription(request.getParameter("description"));
				profForm.setPrice(request.getParameter("price"));
				//创建model，Product,该类需要序列化
				Product product =new Product();
				product.setName(profForm.getName());
				product.setDiscription(profForm.getDiscription());
				try{
					product.setPrice(Float.parseFloat(profForm.getPrice()));
				}catch(Exception e){
				}
				return new ModelAndView("/WEB-INF/jsp/ProductDetail.jsp","product",product);	
			}
		}
3. 这样简单的项目就完成了。下面完善一下项目。

---
## 6.完善上面的项目
**1)完善：**
1. 完善部署描述符（web.xml）：添加springmvc的配置文件
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
		
			<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
			<servlet>
				<servlet-name>springmvc</servlet-name>
				<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
				<init-param>
					<param-name>contextConfigLocation</param-name>
		         	<param-value>/WEB-INF/springmvc-servlet.xml</param-value>
				</init-param>
				<load-on-startup>1</load-on-startup>
			</servlet>
		
			<!-- Map all requests to the DispatcherServlet for handling -->
			<servlet-mapping>
				<servlet-name>springmvc</servlet-name>
				<url-pattern>/</url-pattern>
			</servlet-mapping>
		</web-app>
2. 完善配置类，添加视图解释器bean：
		<?xml version="1.0" encoding="UTF-8"?>
		<beans  
		    xmlns="http://www.springframework.org/schema/beans"  
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xmlns:context="http://www.springframework.org/schema/context"  
		    xmlns:aop="http://www.springframework.org/schema/aop"   
		    xsi:schemaLocation="http://www.springframework.org/schema/beans   
		                        http://www.springframework.org/schema/beans/spring-beans.xsd  
		                        http://www.springframework.org/schema/aop    
		                        http://www.springframework.org/schema/aop/spring-aop.xsd  
		                        http://www.springframework.org/schema/context       
		                        http://www.springframework.org/schema/context/spring-context.xsd">
		 	<bean name="/product_input.action" class="Controller.InputProductController">
		 	</bean>
		 	<bean name="/product_save.action" class="Controller.SaveProductController">
		 	</bean>
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp/"></property>
		 		<property name="suffix" value=".jsp"></property>
		 	</bean>
		 </beans>
3. 将控制器类中的ModelAndView修改为如下格式(去掉前缀与后缀)：
		new ModelAndView("ProductForm")；

**2)测试：**
1. 将项目部署到Tomcat上，在地址栏输入：
		http://localhost:8080/springmvc/product_input.action
2. 跳转到如下界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-06.jpg)
3. 输入后提交条转到详情：
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-07.jpg)

**3)我理解的示意图图：**
![](http://p5ki4lhmo.bkt.clouddn.com/00056SpringMVC%E5%AD%A6%E4%B9%A01-08.jpg)

---