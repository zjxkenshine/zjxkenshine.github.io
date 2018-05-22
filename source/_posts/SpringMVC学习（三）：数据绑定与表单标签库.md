---
title: SpringMVC学习（三）：数据绑定与表单标签库
date: 2018-5-20 21:54:12 
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
	- 数据绑定简介与使用
	- Spring表单标签库
	- 表单标签库使用实例
3. Spring的表单标签库使用的不是很多，所以了解一些就行。
主要的前台页面还是以JSTL和EL表达式为主。

---
## 1.数据绑定的简介与使用
1. 在前面的例子中的控制器类方法中：
		@RequestMapping(value="/product-save")
		public String SaveProduct(ProductForm productForm,RedirectAttributes redirectAttributes){
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
例子中(详见上一节笔记)并没有为ProductForm对象填充任何数据，但是在后面却可以在ProductForm对象中获取到用户输入到表单中的信息。
SpringMVC自动将用户输入到表单中的数据同ProductForm对象绑定，这就是数据绑定。
2. 数据绑定是将用户输入绑定到领域模型的一种特性。
类型为String的HTTP请求参数可以用于填充**各种不同类型**的对象属性。
3. 上面的ProductForm是一个表单类,作用只是为了转换price的类型。
因为数据绑定可以绑定各种类型，所以就不需要表单类了。
4. 上述的代码可以简化如下：
		@RequestMapping(value="/product-save")
		public String SaveProduct2(Product product,RedirectAttributes redirectAttributes){
			long id=productService.add(product);
			redirectAttributes.addFlashAttribute("product",product);
			return "redirect:/print_product/"+id;
		}
5. 以上只是简单的数据绑定，更加精细的数据绑定需要用Spring表单标签库提供的属性来加以控制。

---
## 2.Spring表单标签库
**表单标签库简介与使用**
1. Spring表单标签库包含了可以用在JSP页面中渲染HTML元素的标签。
这些标签在提供原有的HTML标签功能的基础上，还提供了一些和数据绑定相关的属性。
2. Spring的表单标签库和SpringMVC框架是集成在一起的，因此它们可以直接使用命令对象（command object)和其他由控制器处理的数据对象。
3. 在JSP页面中引入表单标签库：
在JSP页面开头处声明
		<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
即可，然后使用`<form:xxx>`来使用各渲染标签。
4. Spring表单标签库的标签：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-01.jpg)
5. 下面详细介绍这些标签的属性及使用。
(spring表单标签库提供的属性都是可选的)

### form,input,password,hidden标签
**1)form表单标签：**
1. 表单标签用于渲染HTML表单。在其中需要使用其他表单标签。
2. form标签有HTML表单的各项属性(action,method等)，还提供了一些额外的属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-02.jpg)
3. 最重要的是commandName标签，定义了模型(Model)属性（键）的名称。
其中包含了一个`backing object`，这个对象的属性将用来填充表单。
如果表单设置了该属性，那么在跳转到该视图(初始化该JSP)的请求方法中一定要有对应的模型属性。
`Model.addAttribute("key",Object);`
4. 下面使用代码说明：
BookAddForm视图中的表单标签：
		...
		<form:form commandName="book" action="book_save" method="post">
		...
		</form>
		...
初始化请求方法：
		@RequestMapping(value="/book_input")
		public String inputBook(Model model){
			...
			Model("book",new Book());	//一定要设置，否则会因表单标签找不到backing object而报错
			return "BookAddForm";
		}
5. 一般来说仍需要使用HTML表单提供的action和method属性。
6. 关于htmlEscape：
是否转义特殊字符，默认为true，若要设置不对特殊字符进行转义，需设置为false。

**2)input标签：**
1. input标签渲染了`<input type="text"/>`元素。
除了input标签原有的属性外，还有以下扩展属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-03.jpg)
2. 其中最重要的是path属性，可以用于将这个标签绑定`backing object`的一个属性。
如：
		<form:form commandName="book" action="book_save" method="post">
			<form:input id="bookname" path="bookname" cssErrorClass="errorBox"/>
		</form>
会被渲染成：
		<input type="text" id="bookname" name="bookname"/>
将该标签和book的bookname属性绑定。
(如果该属性不存在，也不会出错，只是无法进行数据绑定)
3. cssErrorClass属性一般情况下不起作用，只有绑定属性(bookname)的输入验证错误并且使用该表单重新显示用户输入，input标签会被渲染如下：
		<input type="text" id="bookname" name="bookname" class="errorBox"/>
4. 可以绑定嵌套对象：
		<form:input id="authorname" path="author.name"/>
(如果将author对象嵌入在book对象中)
该input标签将绑定到book中的author对象的name属性

**3)password标签：**
1. 渲染了`<input type="password"/>`元素。
和password元素相似，只不过多了一个showPassword属性。
除了HTML原有的属性外，提供了如下的属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-04.jpg)
2. 示例如下：
		<form:password id="pwd" path="password" cssClass="normal"/>
会被渲染成如下的HTML标签：
		<input type="password" name="password" class="normal"/>

**4)hidden标签：**
1. 渲染`<input type="hidden">`元素。
因为不可见，所以没有CSS相关的属性。
除了原有的属性外，提供了如下两个属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-05.jpg)
2. 示例：
		<form:hidden path="bookId"/>
会被渲染成：
		<input type="hidden" name="bookId"/>

### textarea,checkbox(es),radiobutton(s)标签
**1)textarea标签：**
1. 渲染了HTML的textarea元素。
除了原有的属性之外，还提供了以下的属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-06.jpg)
2. 示例：
		<form:textarea path="discripe" tabindex="4" rows="5" cols="80"/>

**2)checkbox标签：**
1. 渲染一个`<input type="checkbox"/>`元素，多选框。
2. 除了原有的属性，还有下列额外属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-07.jpg)
3. 示例：
		<form:checkbox path="booktype" value="type1"/>
		<form:checkbox path="booktype" value="type2"/>
		<form:checkbox path="booktype" value="type3"/>

**3)radiobutton标签：**
1. 渲染一个`<input type="radio"/>`元素，单选框。
2. 除了原有的属性，还有下列额外属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-08.jpg)
3. 示例：
		<form:radiobutton path="sex" value="男"/>
		<form:radiobutton path="sex" value="女"/>

**4)checkboxes标签：**
1. 渲染多个`<input type="checkbox"/>`标签，多选框。
2. 提供了如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-09.jpg)
3. items属性提供了用于生成多选框的数据集合。
itemLabel属性定义了数据集合中的对象的某一属性，用来做字面量。
itemValue属性定义了数据集合中的对象的某一属性，用来做选取得到的值。
4. 示例如下：
		<form:checkboxes path="typeIds" items="${typelist}" itemLabel="typeName" itemValue="typeId"/>

**5)radiobuttons标签：**
1. 渲染了多个`<input type="radio"/>`元素，单选框，用法和属性checkboxes标签类似。
2. 提供了下列属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-10.jpg)
3. 使用示例：
		<form:radiobuttons path="sex" items="${sexlist}"/>

### select,option(s),errors标签
**1)select标签：**
1. 渲染了一个HTML的select元素。
被渲染的元素可以来自items属性，也可以来自option或者options标签。
2. 提供了如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-11.jpg)
3. 使用示例如下(通过items元素绑定数据)
		<form:select id="type" items=${typelist} itemLabel="typeName" itemValue="typeId"/>

**2)option标签：**
1. 渲染了select元素中的一个HTML的option元素。
2. 只提供了一些和css相关的渲染属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-12.jpg)
3. 使用示例：(通过items和option标签绑定数据)
		<form:select id="type" items=${typelist} itemLabel="typeName" itemValue="typeId">
			<form:option value="0">请选择类型</form:option>
		</form:select>
或者也可以使用HTML的option元素：
		<option value="0">请选择类型<option>

**3)options标签：**
1. options标签生成一个HTML的option元素列表。
2. 除了HTML属性外还提供了如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-13.jpg)
3. 使用示例：(option和options绑定数据)
		<form:select path="type">  
			<option/>请选择人员
			<form:options items="${typelist}" itemLabel="typeName" itemValue="typeId"/>  
		</form:select>

**4)errors标签：**
1. 渲染一个或者多个HTML的span元素，每个span元素中都包含一个字段错误。
可以用于显示一个特定字段的错误，或者所有字段的错误。
2. 提供了如下属性：(cssClss和cssStyle未列出)
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-14.jpg)
3. 使用示例：
显示所有字段的错误：
		<form:errors path="*"/>
显示与某个`from bacjing object`对象的属性(author)相关的字段错误：
		<form:errors path="author"/>

---
## 3.数据绑定示例
示例说明：
一个关于书籍管理的简单应用（不涉及数据库层）
有实体类Book,Book中又包含了Catgoty类对象。
有列出书目，添加新书以及编辑数目功能。

### 基本配置及模型类
1. Category类：
		//书的类型
		public class Category implements Serializable{
			private static final long serialVersionUID = 1558731101438330815L;
			private int id;		//类型ID
			private String name;	//类型名
			public Category() {
			}
			public Category(int id,String name) {
				this.id=id;
				this.name=name;
			}
			public int getId() {
				return id;
			}
			public void setId(int id) {
				this.id = id;
			}
			public String getName() {
				return name;
			}
			public void setName(String name) {
				this.name = name;
			}
			@Override
			public String toString() {
				return "Category [id=" + id + ", name=" + name + "]";
			}
		}
2. Book类：
		public class Book implements Serializable{
			private static final long serialVersionUID = 8761144554202637953L;
			private long id;
			private String isbn; //国际标准书号
			private String title; //书名
			private Category category; //类型
			private String author;	//作者
			public Book() {
			}
			public Book(long id, String isbn, String title, Category category,
					String author) {
				super();
				this.id = id;
				this.isbn = isbn;
				this.title = title;
				this.category = category;
				this.author = author;
			}
			//get,set与toString方法
		}
3. 配置文件如下：
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
			<context:component-scan base-package="Controller3"></context:component-scan>
			<context:component-scan base-package="Service3"></context:component-scan>
			<mvc:annotation-driven></mvc:annotation-driven>
			<mvc:resources location="/css/**" mapping="/css"></mvc:resources>
			<mvc:resources location="/*.html" mapping="/"></mvc:resources>
			
		 	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		 		<property name="prefix" value="/WEB-INF/jsp3/"></property>
		 		<property name="suffix" value=".jsp"></property>
		 	</bean>
		 </beans>
4. web.xml文件如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
		
			<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
			<servlet>
				<servlet-name>springmvc</servlet-name>
				<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
				<init-param>
					<param-name>contextConfigLocation</param-name>
				<param-value>/WEB-INF/jsp3/springmvc-servlet.xml</param-value>
				</init-param>
				<load-on-startup>1</load-on-startup>
			</servlet>
		
			<!-- Map all requests to the DispatcherServlet for handling -->
			<servlet-mapping>
				<servlet-name>springmvc</servlet-name>
				<url-pattern>/</url-pattern>
			</servlet-mapping>
			
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
		</web-app>
5. 创建相应的BookController类和BookService接口，基本结构如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-15.jpg)

### 添加与查看书籍功能
**1)初始化添加页面：**
1. BookService接口中添加方法如下：
		public interface BookService {
			List<Category> getAllCategories();
			Category getCategoryById(int id);
		}
2. BookService的实现类BookServiceImpl代码如下：
		@Service
		public class BookServiceImpl implements BookService{
			private List<Category> categories=new ArrayList<Category>();
			private List<Book> booklist=new ArrayList<Book>();
			
			public BookServiceImpl() {
				categories.add(new Category(1,"技术"));
				categories.add(new Category(2,"小说"));
				categories.add(new Category(3,"旅游"));
				categories.add(new Category(4,"生活"));
				categories.add(new Category(5,"历史"));
			}
			
			@Override
			public List<Category> getAllCategories() {
				return categories;
			}
			
			@Override
			public Category getCategoryById(int id) {
				for(Category category:categories){
					if(id==category.getId()){
						return category;
					}
				}
				return null;
			}
		}
3. 控制器类BookController代码如下：
		@Controller
		public class BookController {
			@Autowired
			BookService bookService;
			
			private static final Log logger=LogFactory.getLog(BookController.class);
			//初始化添加书籍视图
			@RequestMapping("/book_input")
			public String inputBook(Model model){
				List<Category> categories=bookService.getAllCategories();
				model.addAttribute("categories",categories);
				model.addAttribute("book",new Book());
				return "AddBook";
			}
		}
4. WEB-INF/jsp3/AddBook.jsp代码如下：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<%@taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>添加书籍</title>
		</head>
		<body>
			<form:form commandName="book" action="book_save" method="post">
				<label for="title">书籍名字：</label>
				<form:input path="title" id="title"/><br/>
				<label for="author">作者：</label>
				<form:input path="author" id="author"/><br/>
				<label for="isbn">国际标准书号(ISBN):</label>
				<form:input path="isbn" id="isbn"/><br/>
				<label for="category">书籍类型：</label>
				<form:select path="category.id" id="category" items="${categories}" itemLabel="name" itemValue="id"/><br/>
				<input type="reset" id="reset">
				<input type="submit" id="submit">
			</form:form>
		</body>
		</html>
5. 测试，初始化成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-16.jpg)
6. 使用Spring表单标签库时出现了一些问题，详见Spring解决方案一问题8。

**2)添加表单处理方法及列表视图：**
1. 添加BookService接口的方法：
		public interface BookService {
			List<Category> getAllCategories();
			Category getCategoryById(int id);
			List<Book> getAllBooks();
			long getNextId();
			Book saveBook(Book book);
		}
2. 在实现类BookServiceImpl中添加实现方法：
		...
		@Override
		public List<Book> getAllBooks() {
			return booklist;
		}
		
		@Override
		public Book saveBook(Book book) {
			book.setId(getNextId());
			booklist.add(book);
			return book;
		}
		
		@Override
		synchronized public long getNextId() {
			long id=0l;
			for(Book book:booklist){
				if(book.getId()>id){
					id=book.getId();
				}
			}
			return id+1;
		}
		...
3. 在BookController控制器类中添加一个处理添加书籍的方法：
添加savaBook处理方法如下：
		@RequestMapping("/book_save")
		public String saveBook(@ModelAttribute Book book){
			logger.info("调用saveBook请求处理方法");
			Category category=bookService.getCategoryById(book.getCategory().getId());
			book.setCategory(category);
			bookService.saveBook(book);
			return "redirect:/book_list";
		}
添加book列表的处理方法如下：
		@RequestMapping("/book_list")
		public String bookList(Model model){
			logger.info("调用bookList请求处理方法");
			List<Book> books=bookService.getAllBooks();
			model.addAttribute("booklist",books);
			return "BookList";
		}
5. 需要用到JSTL，所以添加JSTL的pom.xml依赖：
		<!--old jstl--><!--
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>-->
		
		<!--jstl-->
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-spec</artifactId>
			<version>1.2.5</version>
		</dependency>
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-impl</artifactId>
			<version>1.2.5</version>
		</dependency>
6. 创建BookList.jsp视图：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		 <%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>书籍列表</title>
		</head>
		<body>
		<table>
		<tr>
			<th>书籍名</th>
			<th>作者</th>
			<th>类型</th>
			<th>ISBN</th>
			<th>操作</th>
		</tr>
		<c:forEach items="${booklist}" var="book">
			<tr>
				<td>${book.title}</td>
				<td>${book.author}</td>
				<td>${book.category.name}</td>
				<td>${book.isbn}</td>
				<td><a href="book_edit/${book.id}">编辑</a></td>
			</tr>
		</c:forEach>
		</table>
		</body>
7. 测试：
添加页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-17.jpg)
添加后跳转到书籍列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-18.jpg)
控制台输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-19.jpg)

### 编辑书籍功能
**1)跳转到编辑表单并显示：**
1. 添加BookService接口方法：
		Book getBookById(long id);
2. 添加BookServiceImpl中的实现方法：
		@Override
		public Book getBookById(long id) {
			for(Book book:booklist){
				if(book.getId()==id){
					return book;
				}
			}
			return null;
		}
3. 点击编辑时的请求处理方法（编辑表单的）
在控制器类中添加请求处理方法editBook方法：
		@RequestMapping("/book_edit/{id}")
		public String editBook(Model model,@PathVariable long id){
			logger.info("调用editBook请求处理方法");
			Book book=bookService.getBookById(id);
			List<Category> categories=bookService.getAllCategories();
			model.addAttribute("categories",categories);
			model.addAttribute("book",book);
			return "EditBook";
		}
4. 创建EditBook.jsp视图：
		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<%@taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>编辑书籍</title>
		</head>
		<body>
			<!--注意这里的action地址需要使用绝对路径，否则会变成/book_edit/book_update,或者也可以修改@RequestMapping-->
			<form:form commandName="book" action="/springmvc/book_update" method="post">
				<form:hidden path="id"/>
				<label for="title">书籍名字：</label>
				<form:input path="title" id="title"/><br/>
				<label for="author">作者：</label>
				<form:input path="author" id="author"/><br/>
				<label for="isbn">国际标准书号(ISBN):</label>
				<form:input path="isbn" id="isbn"/><br/>
				<label for="category">书籍类型：</label>
				<form:select path="category.id" id="category" items="${categories}" itemLabel="name" itemValue="id"/><br/>
				<input type="reset" id="reset">
				<input type="submit" id="submit">
			</form:form>
		</body>
		</html>
5. 测试：
添加书籍并跳转到书籍列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-20.jpg)
点击编辑：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-21.jpg)
控制台输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-22.jpg)

**2)更新书籍信息：**
1. 添加BookService接口方法：
		void updateBook(Book book);
2. 添加BookServiceImpl类实现方法：
		@Override
		public void updateBook(Book book) {
			int count=booklist.size();
			for(int i=0;i<count;i++){
				Book book1=booklist.get(i);
				if(book1.getId()==book.getId()){
					booklist.set(i,book);
				}
			}
		}
3. 添加控制器请求处理方法：
		//更新书籍处理方法
		@RequestMapping("/book_update")
		public String updateBook(Book book){
			logger.info("调用saveBook请求处理方法");
			Category category=bookService.getCategoryById(book.getCategory().getId());
			book.setCategory(category);
			bookService.updateBook(book);
			return "redirect:/book_list";
		}
4. 测试：
更新前列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-24.jpg)
点击编辑：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-25.jpg)
点击提交，更新后列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-23.jpg)
控制台输出：
![](http://p5ki4lhmo.bkt.clouddn.com/00065SpringMVC%E5%AD%A6%E4%B9%A03-26.jpg)

---
## 4.测试代码总结
1. 接口BookService代码：
		public interface BookService {
			List<Category> getAllCategories();
			Category getCategoryById(int id);
			List<Book> getAllBooks();
			long getNextId();
			Book saveBook(Book book);
			Book getBookById(long id);
			void updateBook(Book book);
		}
2. 接口实现类BookServiceImpl代码：
		@Service
		public class BookServiceImpl implements BookService{
			private List<Category> categories=new ArrayList<Category>();
			private List<Book> booklist=new ArrayList<Book>();
			
			public BookServiceImpl() {
				categories.add(new Category(1,"技术"));
				categories.add(new Category(2,"小说"));
				categories.add(new Category(3,"旅游"));
				categories.add(new Category(4,"生活"));
				categories.add(new Category(5,"历史"));
			}
		
			@Override
			public List<Category> getAllCategories() {
				return categories;
			}
		
			@Override
			public Category getCategoryById(int id) {
				for(Category category:categories){
					if(id==category.getId()){
						return category;
					}
				}
				return null;
			}
		
			@Override
			public List<Book> getAllBooks() {
				return booklist;
			}
		
			@Override
			public Book saveBook(Book book) {
				book.setId(getNextId());
				booklist.add(book);
				return book;
			}
		
			@Override
			synchronized public long getNextId() {
				long id=0l;
				for(Book book:booklist){
					if(book.getId()>id){
						id=book.getId();
					}
				}
				return id+1;
			}
		
			@Override
			public Book getBookById(long id) {
				for(Book book:booklist){
					if(book.getId()==id){
						return book;
					}
				}
				return null;
			}
		
			@Override
			public void updateBook(Book book) {
				int count=booklist.size();
				for(int i=0;i<count;i++){
					Book book1=booklist.get(i);
					if(book1.getId()==book.getId()){
						booklist.set(i,book);
					}
				}
			}
		}
3. 控制器类BookController代码：
		package Controller3;
		
		import java.util.List;
		import org.apache.commons.logging.Log;
		import org.apache.commons.logging.LogFactory;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Controller;
		import org.springframework.ui.Model;
		import org.springframework.web.bind.annotation.ModelAttribute;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RequestMapping;
		import Model.Book;
		import Model.Category;
		import Service3.BookService;
		import Service3.BookServiceImpl;
		
		@Controller
		public class BookController {
			@Autowired
			BookService bookService;
			
			private static final Log logger=LogFactory.getLog(BookController.class);
			//初始化添加书籍页面
			@RequestMapping("/book_input")
			public String inputBook(Model model){
				logger.info("调用inputBook请求处理方法");
				List<Category> categories=bookService.getAllCategories();
				model.addAttribute("categories",categories);
				model.addAttribute("book",new Book());
				return "AddBook";
			}
			//添加书籍处理
			@RequestMapping("/book_save")
			public String saveBook(@ModelAttribute Book book){
				logger.info("调用saveBook请求处理方法");
				Category category=bookService.getCategoryById(book.getCategory().getId());
				book.setCategory(category);
				bookService.saveBook(book);
				return "redirect:/book_list";
			}
			//书籍列表初始化处理方法
			@RequestMapping("/book_list")
			public String bookList(Model model){
				logger.info("调用bookList请求处理方法");
				List<Book> books=bookService.getAllBooks();
				model.addAttribute("booklist",books);
				return "BookList";
			}
			//编辑表单初始化
			@RequestMapping("/book_edit/{id}")
			public String editBook(Model model,@PathVariable long id){
				logger.info("调用editBook请求处理方法");
				Book book=bookService.getBookById(id);
				List<Category> categories=bookService.getAllCategories();
				model.addAttribute("categories",categories);
				model.addAttribute("book",book);
				return "EditBook";
			}
			//更新书籍处理方法
			@RequestMapping("/book_update")
			public String updateBook(Book book){
				logger.info("调用saveBook请求处理方法");
				Category category=bookService.getCategoryById(book.getCategory().getId());
				book.setCategory(category);
				bookService.updateBook(book);
				return "redirect:/book_list";
			}
		}

---