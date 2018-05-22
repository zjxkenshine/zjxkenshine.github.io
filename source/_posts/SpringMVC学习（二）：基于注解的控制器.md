---
title: SpringMVC学习（二）：基于注解@Controller的控制器
date: 2018-5-9 15:59:23
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
	- @Controller注解和@RequestMapping注解
	- 请求方法的参数类型及返回值类型(请求处理的方法编写)
	- 如何应用基于注解的控制器
	- 重定向与flash属性
	- 请求参数与路径变量
	- @ModelAttribute注解

---
## 1.Spring MVC注解类型
**1)使用注解控制器的好处：**
1. 一个注解类型的控制器可以处理多个动作。
（实现Conrtoller接口的控制器只能处理一个动作）
这样就可以将相关的多个动作写在一个控制器类中从而减少应用程序中类的数量。
2. 基于注解的控制器的请求不需要写在配置文件中，
使用@RequestMapping注解可以对一个类或者方法进行请求处理。
3. Spring MVC中最重要的注释类型：
@Controller和@RequestMapping

**2)@Controller注解类型：**
1. org.springframework.stereotype.Controller注解类（@Controller可以标识这个类是一个注解类）
2. 使用@Controller注解主要有三步：
	- 为控制器类添加@Controller注解
	- 在springmvc-servlet.xml`<beans>`标签中声明Spring上下文(一般都有)
	- springmvc-servlet.xml中使用`<context:component-scan>`标签自动扫描注解类
3. 具体步骤：
添加@Controller注解：
		package Controller2;
		import org.springframework.stereotype.Controller;
		@Controller
		public class ProductController {
			...
		}
声明上下文：
		<beans
		...
		xmlns:context="http://www.springframework.org/schema/context"
		...>
使用component-scan标签自动扫描注解类：
		...
		<context:component-scan base-package="Controller2"></context:component-scan>
		...
4. 最好确保所有的控制器都在一个包下，避免扫描一些不需要的bean。

**3)@RequestMapping注解类型：**
1. 配置完控制器类之后，需要为每一个动作(action)开发相应的处理方法，这时就需要使用@RequestMapping注解了。
		org.springframework.web.bind.annotation.RequestMapping
2. @RequestMapping：可以映射一个请求和一种方法
@RequestMapping可以修饰控制器类或者其中的方法。

**4)@RequestMapping注解修饰控制类方法：**
1. 一个添加了@RequestMapping注解的控制器类方法将会变成一个处理请求的方法。
2. 简单的使用@RequestMapping注解方法：
		@Controller
		public class ProductController {
				@RequestMapping(value="/product-input")
				public String ProductInput(){
					return null;
				}
		}
value属性将URI映射到方法上，这时可以通过以下URL访问该方法：
		http://域名/context/product-input
context是应用名，我这里是springmvc
3. value属性是@RequestMapping的默认属性：
		@RequestMapping("product-input")
相当于：
		@RequestMapping(value="product-input")
可以是空字符串，则被映射到如下路径：
		http://域名/context
4. method属性可以指定该方法仅处理那些HTTP请求方法。
HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。
HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。
如果没有指定
5. method属性示例：
指定处理一种请求方法：
		@RequestMapping(value="/product-input",method=RequestMethod.POST)	//也可以加大括号
		public String ProductInput(){..}
指定处理多种请求方法则需要加大括号：
		@RequestMapping(value="/product-input",method={RequestMethod.POST，RequestMethod.GET})
		public String ProductInput(){..}
6. SpringMVC还为每一种请求提供了相关的映射注解，如：
		@PostMapping(value="")
		@GetMapping(value="")
		@DeleteMapping(value="")
		@PutMapping(value="")
		...

**5)@RequestMapping注解修饰控制类：**
1. 将@RequestMapping放在控制类的类级别上修饰，则所有方法的请求都会映射为相对于类级别的请求。
2. 示例：
		@Controller
		@RequestMapping(value="/product")
		public class ProductController {
				@RequestMapping(value="/product-input")
				public String ProductInput(){
					return null;
				}
		}
则访问ProductInput()方法需要使用如下URL：
		http://域名/context/product/product-input
3. 对不同控制器类使用不同的@RequestMapping类级别注解可以方便区分不同的控制器类。

---
## 2.请求处理的方法编写
**1)简介：**
1. 上面的两个注解的使用可以让我们声明并且将URL映射到正确的请求处理方法，而请求处理方法的编写也有一定的规范。
2. 每个请求方法可以有多个不同类型的参数，以及一个多钟类型的返回值结果。
3. 书上的简单示例：
例一：使用session做参数
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-01.jpg)
例二：request及locale做参数
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-02.jpg)

**2)请求处理方法的参数：**
以下是可以在请求处理方法中出现的参数：
1. Request,Response及Session：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-03.jpg)
2. 其他多种类型：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-04.jpg)

**3)请求处理方法的返回值：**
返回值可以是以下类型的对象：
- ModelAndView
- Model
- Map包含模型的属性
- View
- 代表逻辑视图名的String
- void
- Callable
- DeferredResult
- 提供对Servlet的访问，以响应Http头部和内容的HttpEntity和ResponseEntity对象
- 其他任意类型，Spring会将其视作输出给View的对象模型。

---
## 3.基于注解控制器的简单使用
**1)配置文件：**
1. 部署描述符(web.xml文件)：
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
		
			<servlet>
				<servlet-name>springmvc</servlet-name>
				<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
				<init-param>
					<param-name>contextConfigLocation</param-name>
					<param-value>/WEB-INF/jsp2/springmvc-servlet.xml</param-value>
				</init-param>
				<load-on-startup>1</load-on-startup>
			</servlet>
		
			<servlet-mapping>
				<servlet-name>springmvc</servlet-name>
				<url-pattern>/</url-pattern>
			</servlet-mapping>
		</web-app>
url-pattern设置为/说明所有的请求(包括对静态资源的请求)都将被映射到dispatcher servlet。
2. springmvc-servlet.xml配置文件如下：
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
			<context:component-scan base-package="Controller2"></context:component-scan>
			<!--静态资源调用-->
			<mvc:annotation-driven></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			<!--配置视图解析器-->
			<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
				<property name="prefix" value="/WEB-INF/jsp2/"></property>
				<property name="suffix" value=".jsp"></property>
			</bean>
		</beans>
需要声明spring-mvc命名空间：
		<beans
		...
		xmlns:mvc="http://www.springframework.org/schema/mvc">
3. 调用静态资源：
			<mvc:annotation-driven></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
`<mvc:annotation-driven/>`标签：
注册用于支持基于注解的控制器的请求处理方法的bean对象。
`<mvc:resources>`标签：
只是SpringMVC的哪些静态资源需要单独处理
需要注意的两点：
	- 如果没有`<mvc:annotation-driven>`标签则`<mvc:resources>`标签会阻止任意控制器被调用。
	- 如果没有使用`<mvc:resources>`标签则不需要使用`<mvc:annotation-driven>`标签。(不访问静态资源)

**2)Controller控制器类：**
1. 一个控制器类可以包含多个请求处理方法：(实体类就用原来的)
		package Controller2;
		
		import org.apache.commons.logging.Log;
		import org.apache.commons.logging.LogFactory;
		import org.springframework.stereotype.Controller;
		import org.springframework.ui.Model;
		import org.springframework.web.bind.annotation.RequestMapping;
		import Model.Product;
		import Model.ProductForm;
		
		@Controller
		public class ProductController {
			private static final Log logger=LogFactory.getLog(ProductController.class);
				
			@RequestMapping(value="/product-input")
			public String ProductInput(){
				logger.info("调用ProductInput方法处理请求");
				return "ProductForm"; //相当于访问/WEB-INF/jsp2/ProductForm.jsp
			}
			
			@RequestMapping(value="/product-save")
			public String SaveProduct(ProductForm productForm,Model model){
				//不用再创建ProductForm了
				logger.info("调用SaveProduct方法处理请求");
				Product product =new Product();
				product.setName(productForm.getName());
				product.setDiscription(productForm.getDiscription());
				try{
					product.setPrice(Float.parseFloat(productForm.getPrice()));
				}catch(Exception e){
				}
				model.addAttribute("product",product);	//相当于request.setAttribute,可以通过el表达式访问
				return "ProductDetails";
			}
		}
2. 第二参数的org.springframework.ui.Model类对象。
无论是否使用Spring MVC都会在调用请求处理方法时创建一个Model实例，用于增加需要显式在视图中的属性。可以通过addAttribute方法来添加：
		model.addAttribute("product",product);

**3)视图：**
1. /WEB-INF/jsp2/ProductForm.jsp页面代码如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>Product Form</title>
		</head>
		<body>
		<form action="product-save" method="post">
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
2. /WEB-INF/jsp2/ProductDetails.jsp页面代码如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
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

**4)测试：**
1. 访问http://localhost:8080/springmvc/product-input，到达如下界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-05.jpg)
2. 输入信息后提交，跳转到以下界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-06.jpg)
3. 控制台输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-07.jpg)
4. 需要productForm中的属性和ProductForm.jsp表单的name对应，否则某些属性会丢失。

---
## 4.使用@Autowired和@Service进行依赖注入
**1)简单的使用过程：**
1. 在使用J2EE的时候通常是使用导包的形式得到Service实例化对象，使用Spring之后可以使用@Autowired来自动注入。
2. 使用@Service，@Controller,@Component注解标注的类都会被自动扫描到。
3. 可以在Service业务层对得到的结果做一些处理。

**2)简单测试：**
1. 添加ProductService业务接口：
		package Service2;
		import Model.Product;
		public interface ProductService {
			long add(Product product);
			Product print(long id);
		}
2. 实现类ProductServiceImpl实现类如下：(一个统计并输出商品的功能)
		package Service2;
		
		import java.util.HashMap;
		import java.util.Map;
		import java.util.concurrent.atomic.AtomicLong;
		import org.springframework.stereotype.Service;
		import Model.Product;
		@Service
		public class ProductServiceImpl implements ProductService{
			private Map<Long,Product> products=new HashMap<Long, Product>();
			private AtomicLong productId=new AtomicLong();
			@Override
			public long add(Product product) {
				Long newId=productId.incrementAndGet();
				products.put(newId, product);
				return newId;
			}
			@Override
			public Product print(long id) {
				Product product=products.get(id);
				System.out.println(product);
				return product;
			}
		}
3. 修改控制类代码如下：
		package Controller2;
		
		import org.apache.commons.logging.Log;
		import org.apache.commons.logging.LogFactory;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Controller;
		import org.springframework.ui.Model;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RequestMapping;
		import Model.Product;
		import Model.ProductForm;
		import Service2.ProductService;
		@Controller
		public class ProductController {
			private static final Log logger=LogFactory.getLog(ProductController.class);
			@Autowired
			ProductService productService;
				
			@RequestMapping(value="/product-input")
			public String ProductInput(){
				logger.info("调用ProductInput方法处理请求");
				return "ProductForm"; //相当于访问/WEB-INF/jsp2/ProductForm.jsp
			}
			
			@RequestMapping(value="/product-save")
			public String SaveProduct(ProductForm productForm){
				//不用再创建ProductForm了
				logger.info("调用SaveProduct方法处理请求");
				Product product =new Product();
				product.setName(productForm.getName());
				product.setDiscription(productForm.getDiscription());
				try{
					product.setPrice(Float.parseFloat(productForm.getPrice()));
				}catch(Exception e){
				}
				long id=productService.add(product);
				return "redirect:/print_product/"+id;	//重定向到下面的处理方法
			}
			
			@RequestMapping(value="/print_product/{id}")
			public String PrintProduct(@PathVariable long id,Model model){
				Product product = productService.print(id);
				model.addAttribute("product",product);	//相当于request.setAttribute
				return "ProductDetails";
			}
		}
4. 在springmvc-servlet.xml配置文件中配置自动扫描Service2包：
		......
		<context:component-scan base-package="Controller2"></context:component-scan>
		<context:component-scan base-package="Service2"></context:component-scan>
		......
5. 测试：
添加商品：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-08.jpg)
提交：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-09.jpg)
控制台输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-10.jpg)

---
## 5.重定向和Flash属性
**1)重定向与请求转发的简单区别：**
1. 控制器return时默认的是请求转发。使用如下方式则是重定向
		return "redirect:/print_product/"
2. 请求转发比重定向快，因为重定向是经过客户端的，而请求转发则没有。
(这就导致Model中设置的值在一次重定向之后会消失)
3. 如果要访问其他应用(外部网站)，则只能使用重定向。
4. 重定向可以避免用户在重新刷新一个页面的时候进行重复操作(尤其是对数据库的重复操作)
就像上面的例子那样(有待完善，仅仅这样的话仍然可以通过修改重定向参数而修改显示的数据)
不过可以防止在用户刷新的时候SaveProduct处理方法被二次调用。
5. 重定向不方便的地方就是会将地址栏显露在外面，而且无法轻松传值给目标页面。
(因为经过客户端Model中设置的值在一次重定向之后会消失),Spring3.1版本提供了一个Flash属性来解决这一问题。(详细见后面的例子完善)
		RedirectAttributes.addFlashAttribute("","");

**2)完善上面的例子：**
1. 修改控制器类代码如下：
		package Controller2;
		
		@Controller
		public class ProductController {
			private static final Log logger=LogFactory.getLog(ProductController.class);
			@Autowired
			ProductService productService;
			
			@RequestMapping(value="/product-input")
			public String ProductInput(){
				logger.info("调用ProductInput方法处理请求");
				return "ProductForm"; //相当于访问/WEB-INF/jsp2/ProductForm.jsp
			}
			@RequestMapping(value="/product-save")
			public String SaveProduct(ProductForm productForm,RedirectAttributes redirectAttributes){
				//不用再创建ProductForm了
				logger.info("调用SaveProduct方法处理请求");
				Product product =new Product();
				product.setName(productForm.getName());
				product.setDiscription(productForm.getDiscription());
				try{
					product.setPrice(Float.parseFloat(productForm.getPrice()));
				}catch(Exception e){
				}
				long id=productService.add(product);
				redirectAttributes.addFlashAttribute("product",product);
				return "redirect:/print_product/"+id;	//重定向到下面的处理方法
			}
			
			@RequestMapping(value="/print_product/{id}")
			public String PrintProduct(@PathVariable long id){
				productService.print(id);
				return "ProductDetails";
			}
		}
2. 使用Flash属性则在springmvc-servlet.xml配置文件中一定要添加`mvc:annotation-driven/>`标签,否则会失效。
3. 光是这样有可能会出现乱码的问题，(上面的例子也会)，一个一劳永逸的办法，在web.xml中添加Filter（编码过滤器）：	
		<filter>  
			<filter-name>characterEncodingFilter</filter-name>  
			<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
			<init-param>  
				<param-name>encoding</param-name>  
				<param-value>UTF-8</param-value>  
			</init-param>  
			<init-param>  
				<param-name>forceEncoding</param-name>  
				<param-value>true</param-value>  
			</init-param>  
		</filter>  
		<filter-mapping>  
			<filter-name>characterEncodingFilter</filter-name>  
			<url-pattern>/*</url-pattern>  
		</filter-mapping>
且将jsp页面的编码设置为UTF-8(一般都是)：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
			pageEncoding="UTF-8"%>
4. 测试结果：(中文乱码的问题解决了)
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-11.jpg)
刷新该页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-12.jpg)
信息不见了，说明并没有再次执行SaveProduct方法。
(如果是查询数据的话还是使用缓存来再次查询吧)
5. 最好将信息封装以后再传值，否则向上面一样只要修改最后的1就可以控制控制台的输出。

---
## 6.请求参数和路径变量
请求参数与路径变量都可以用于发送值给服务器。二者都是URL的一部分。

**1)请求参数：**
1. 请求参数一般在`?`后面，以key=value的形式存在，以`&`相连接。
如：
		http://local:8080/app04b/product_retrieve?productId=3
2. 如何取值：@RequestParam注解
传统的servlet编程中可以使用如下方式：
		String productId=request.getParamater("productId");
在Spring MVC中可以使用@RequestParam来注释方法参数，如下：
		public void saveProduct(@RequestParam int productId);
这样可以直接在参数获取值。

**2)路径变量：**
1. 和请求参数一样，路径变量也可以用于传值，只是路径变量只有一个值，且使用`/`来区分。
2. 如上面例子的：
		/print_product/{id}
3. 取值方式：@PathVariable注解
		public void saveProduct(@PathVariable int productId);
当方法被调用时URL的id值将会被赋值到路径变量中且可以在方法中使用。
路径变量的值可以是非字符串类型。
4. 可以使用多个路径变量：
		/print_product/{userId}/{roleId}

**3)使用路径变量时会出现的问题：**
1. 能正确解析的路径：
		http://example.com/context/abc
此时正确解析context是上下文，abc是某一路径。
2. 如果URL如下：
		http://example.com/abc/1
想使用默认的上下文应用（abc为路径，1为路径变量）,但是服务器会把abc解析为上下文，而不是默认的上下文。
3. 解决方案，JSTL的URL标记：
书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00059SpringMVC%E5%AD%A6%E4%B9%A02-13.jpg)

---
## 7.@ModelAttribute注解
该注解和model.addAttribute()的作用相同。
有两种用法：修饰参数与修饰方法。

**1)用法一：可以用来访问Model类型的参数**
1. 使用@ModelAttribute注解修饰方法的参数，会将该参数加入到Model参数中。
2. 例子：
		@RequestMapping(value="/product-input")
		public String ProductInput(@ModelAttribute("newProduct") Product product,Model model){
调用该方法时product对象将会加入到model对象中，键为newProduct，如果未指定键值(括号中的值)，则键为默认的类的对象名(product)

**2)用法二：标注一个非请求的处理方法**
1. Spring MVC在调用某一控制类请求处理方法时会先调用该控制类中带有@ModelAttribute注解修饰的非请求的处理方法。
2. 使用@ModelAttribute注解修饰的方法需要满足以下两点：
	- 要么返回值不为空
	- 要么返回值为空，手动添加值到model
3. 返回值不为空：
自行将返回的对象添加到Model中，键值为对象名。
4. 返回值为空：void
需要在该方法参数中添加一个Model参数，并手动使用
`model.addAttribute()`
将某个对象添加到model中。

---