---
title: SpringMVC学习（五）：JSTL（JSP标准标签库）
date: 2018-5-24 22:17:10 
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
2. 参考网址：
<http://www.runoob.com/jsp/jsp-jstl.html>
<https://blog.csdn.net/qq_25827845/article/details/53311722>
3. 简单目录：
	- JSTL简介
	- JSTL库
	- 一般标签(行为)
	- 条件标签
	- 遍历标签
	- 与URL相关的标签
	- 格式化行为(标签)
	- SQL相关的标签
	- JSTL函数

---
## 1.JSTL简介及简单使用
**1)简介：**
1. JSTL：
JavaServer Pages Standard Tag Library
JSP标准标签库集合(多个库)
2. JSTL可以用来解决像遍历map或集合，条件测试，xml处理，甚至数据访问或数据操作等常见问题。
3. 这里只介绍JSTL的一些重要标签，完整的标签说明可以参考JSTL规范文档。

**2)简单使用：**
1. 可以下载相关的jar包
官方下载地址：<http://archive.apache.org/dist/jakarta/taglibs/standard/binaries/>
2. 也可以通过添加Maven依赖的方式
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
或者用以下依赖：(上面的是这个的封装)
		<dependency>
			<groupId>javax.servlet.jsp.jstl</groupId>
			<artifactId>jstl-api</artifactId>
			<version>1.2</version>
			<exclusions>
				<exclusion>
					<groupId>javax.servlet</groupId>
					<artifactId>servlet-api</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.servlet.jsp</groupId>
					<artifactId>jsp-api</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
			<groupId>org.glassfish.web</groupId>
			<artifactId>jstl-impl</artifactId>
			<version>1.2</version>
			<exclusions>
				<exclusion>
					<groupId>javax.servlet</groupId>
					<artifactId>servlet-api</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.servlet.jsp</groupId>
					<artifactId>jsp-api</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.servlet.jsp.jstl</groupId>
					<artifactId>jstl-api</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
3. 在JSP页面中根据要使用的标签库来导入相关的库。
如使用JSTL核心库：
		<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
4. 后面所涉及到的body content，正文主体，其实就是jsp的内容。

---
## 2.JSTL标签库
1. JSTL是标准标签库的集合，通过各种标签库来暴其行为。
2. JSTL的标签库大致可以分为5类：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-01.jpg)
3. 在JSP中使用JSTL标签库需要使用taglib命令：
		<%@taglib prefix="前缀" uri="URI地址" %>
前缀可以任意指定，但是最好遵循一般规范。
4. 例如使用函数库：
		<%@taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
5. 每个标签库中有很多标签，每个标签又有很多属性。
	- 书上使用星号`*`来表示必需的关键属性。
	- 用加号`+`表示该属性的值可以是静态字串，El表达式或其他。
	- 未加加号`+`则表示该属性只能是静态字符串。
6. 下面对各个库中的重要标签进行详细介绍。

---
## 3.一般行为(标签)
1. 主要介绍三个标签：
	- out标签
	- set标签
	- remove标签
2. 这是核心库中用于操作有界变量的三个标签。
3. 需要在JSP中引入核心库：
`<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`

**1)out标签：**
1. out标签用于输出表达式结果。
2. out标签属性如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-02.jpg)
3. 有两种使用方式：
无body content（正文内容）：
		<c:out value="value" escapeXml="true/false" default="defaultValue"/>
有body content：
		<c:out value="value" escapeXml="true/false">
			defaultValue
		</c:out>
escapeXml属性默认值为true。
如果value属性值为空则使用default属性的值。
4. 配合EL表达式：
		<c:out value="${sessionScope.name}" escapeXml="true" default="${applicationScope.name}"/>

**2)set标签：**
1. set标签有以下作用：
	- (1)创建一个字符串和一个引用该字符串的有界变量
	- (2)创建一个引用现存有界对象的有界变量
	- (3)设置有界对象的属性
2. set标签属性如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-03.jpg)
创建的变量默认是page范围的，可以使用scope属性来设置。
set标签的语法有四种使用形式。
3. 使用方式一：
		<c:set var="varName" value="varValue" scope="page/request/session/application"/>
如要赋值`name=keshine`可以这么写：
		<c:set var="name" value="keshine" scope="session"/>
4. 使用方式二：通过page content来给变量赋值
		<c:set var="varName" scope="page/request/session/application">
			varValue
		</c:set>
正文body content中可以有JSP代码
5. 使用方式三：设置对象的属性值
		<c:set target="目标对象" property="属性名" value="值"/>
如给对象product的name属性赋值computer：
		<c:set target="${product}" property="name" value="computer"/>
6. 使用方式四：通过page content设置对象的属性值
		<c:set target="目标对象" property="属性名">
			值
		</c:set>
如上面的例子可以这样写：
		<c:set target="${product}" property="name">
			computer
		</set>

**3)remove标签：**
1. 用于删除指定的有界变量(根据变量名以及范围)
2. 有如下两个属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-04.jpg)
3. 如果只是删除了对象的引用，不会将对象也删除，可以通过其他引用访问该对象。
4. 简单使用：
		<c:remove var="name" scope="page">

---
## 4.条件行为(标签)
1. 同样也是核心库中的标签。
通过`<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`导入。
2. 主要介绍两组标签：
	- if标签
	- choose,when,otherwise标签组合

**1)if标签：**
1. 对某个条件进行测试，如果结果为True,就处理body content
没有else标签，所以只能用条件相反的if标签来处理`if...else...`的逻辑
2. 有如下的属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-05.jpg)
3. 两种使用方式：
无页面正文：
		<c:if test="条件" var="varName" scope="范围">
		<c:if test="varName">...</c:if>
生成了一个varName的布尔型变量，可以在后文使用。
有页面正文：
		<c:if test="varName" var="varName" scope="范围">
			需要处理的正文
		</c:if>

**2)choose,when,otherwise标签组合**
1. 相当于Java中的switch,case,default
2. 可以只使用choose,when标签组合，也可以使用这三个标签的组合
3. choose标签和otherwise标签没有属性，when标签必须要有一个test属性。
4. 简单使用
		<c:choose>
			<c:when test="条件一">
				body content1
			</c:when>
			<c:when test="条件二">
				body content2
			</c:when>
			<c:otherwise>
				都不满足时的正文
			</c:otherwise>
		</c:choose>

---
## 5.遍历行为(标签)
也是JSTL核心库中的标签。
主要有标签进行遍历：
- forEach标签
- forToken标签

**1)forEach标签：**
1. 该标签会无数次反复遍历body content或者指定对象集合。
可以遍历的内容包括：
	- Collection,Map的所有实现以及对象数组或者主类型
	- Iterator，Enumeration，但是尽量少用这两个类
2. forEach标签有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-06.jpg)
3. 有两种使用方式：
方式一：固定次数重复body content
		<c:forEach var="变量名" begin="开始" end="结束" step="步长">
			循环主体bodycontent
		</c:forEach>
方式二：遍历集合对象和body content(固定次数,不约束就是全部遍历)
		<c:forEach items="集合对象" [var="变量名" begin="开始" end="结束" step="步长"]>
			循环主体
		</c:forEach>
4. 方式二在SpringMVC学习笔记三中就已经使用过了：
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
5. 关于var属性：
每次循环都会生成一个有界变量，变量中保存的就是一次循环出来的内容，由var属性定义变量名字。上面的循环变量就是book,存储的是book对象。
6. 关于varStatus属性：
存储这个forEach的循环信息。使用该属性定义循环信息的名称。循环信息保存在一个接口对象内，该对象有一个count属性，可以通过`名称.count`获取当前的循环次数。
如：
		<c:forEach items="${booklist}" var="book" varStatus="status">
			<c:out value="${status.count}"/>
		</c:forEach>
7. 可以嵌套forEach标签达到多重循环的效果。

**2)forTokens标签：**
1. 用于隔开以特定分隔符隔开的字符串，相当于Java中的Split各开后再遍历。
使用如下：
		<c:forTokens items="token字符串" delims="分隔符" var="变量名" begin="开始" end="结束" step="步长">
			循环体body content
		</c:forTokens>
2. 属性列表如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-07.jpg)
3. 使用示例：
如下遍历:
		<c:forTokens items="篮球,足球,排球" var="ball" delims=",">
			<c:out value="${ball}"/>
		</c:forTokens>
输出结果是：
		篮球
		足球
		排球

---
## 6.与URL相关的行为
主要介绍url标签及简单介绍redirect标签。

**1)url标签：**
1. url标签作用简单来说就是组合正确的URL
(使URL的解析是正确的)
url标签也是JSTL核心库中的标签。
2. url标签执行如下操作：
	- 如果当前上下文路径为"/"（默认上下文）,则将空字符串附加给指定路径
	- 如果当前上下文路径不为"/",则将上下文路径附加给指定路径
3. 下面以例子说明：
如有应用context为默认上下文，其中有图片资源image/pic.jpg，想要在img标签中调用该图片资源。
正确的url:
		http://host/context/image/pic.jpg
4. 方式一：相对路径
		<img src="image/pic.jpg">
此时通过不同路径访问到的img解析得到的url也不同。
`http://host/context/main/main.jsp`
就会错误解析为：
		http://host/context/main/image/pic.jpg
5. 方式二：绝对路径
		<img src="/image/pic.jpg">
看上去很好，但是通过默认上下文就会出错
如：
`http://host/main/main.jsp`
就会解析为：
		http://host/main/image/pic.jpg
6. 方式三：使用url标签
		<img src="<c:url value="/image/pic.jpg"/>">
具体处理就是是默认上下文加空字符串，不是就加上下文。
7. 方式四：使用EL表达式(手动实现)
		<c:set var="cp" value="${pageContext.request.contextPath}">
		<img src="${cp="/"?" ": cp}/image/pic.jpg"/">
8. 只要记住在需要使用url资源的时候使用`<c:url value="绝对路径"/>`就不太会出错。

**2)redirect标签：**
1. 会发出跳转到客户端的HTTP
2. 有两个属性：
	- url：要跳转到的url
	- context：跳转到外部上下文URL资源时的上下文名称

---
## 7.格式化行为(标签)
1. 格式化标签就是用来对数据进行格式化的，
格式化标签来自国际化标签库，需要如下代码进行导入：
		<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
2. 主要介绍如下标签：
	- formatNumber标签
	- formatDate标签
	- timeZone标签
	- setTimeZone标签
	- parseNumber标签
	- parseDate标签

**1)formatNumber标签：**
1. 用于格式化数字。
该标签可以根据需要，利用它的属性来获得自己想要的格式。
2. 该标签有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-12.jpg)
3. 有关两种使用形式：
无body content：(格式化value属性的值)
		<fmt:formatNumber value="源value" 各项属性/>
有body content：(格式化body content的值)
		<fmt:formatNumber 各项属性>
			要格式化的字符串或者数字
		</fmt:formatNumber>
各属性可选值：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-15.jpg)
4. 最常见的用法是将数字格式转换成货币，使用currencyCode属性来定义一个ISO4217码即可，ISO4217货币代码表如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-13.jpg)
5. 使用示例：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-11.jpg)

**2)formatDate标签：**
1. formatDate标签用于格式化日期。
2. 有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-09.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-10.jpg)
3. 关与timeZone属性的可能值查看timeZone标签部分。
并不是简单的1234。
4. 使用方式：
		<fmt:formatDate value="date" [各项属性]/>
各属性可选值：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-14.jpg)
5. 简单示例：
格式化变量now引用的Date类对象：
		<fmt:formatDate value="${now}" dateStyle="[default|short|...]" />
格式化时间：(Type默认值应该是Date)
		<fmt:formatDate value="${now}" type="time" timeStyle="[default|short|...]" />
格式化日期和时间：
		<fmt:formatDate value="${now}" type="both" dateStyle="[default|short|...]" timeStyle="[default|short|...]" />
格式化带时区的时间：
		<fmt:formatDate value="${now}" type="time" timeZone="CT" />
定制模式的日期和时间：
		<fmt:formatDate value="${now}" type="both" pattern="yyyy-MM-dd" />

**3)timeZone标签：**
1. timeZone标签用于定义时区，格式化body content的值。
用法如下：
		<fmt:timeZone value="时区值" >
			body content
		</fmt:timeZone>
2. 关于时区值：
可以使用`GMT+-n:00`来表示在哪个时区（GMT:国际标准时间）
中国为：`GMT+08:00`
也可以使用时区缩写(自行百度，好像没有中国的)

**4)setTimeZone标签：**
1. setTimeZone标签用于将时区保存在一个变量或者时间配置变量中。
2. 属性如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-16.jpg)
3. 用法如下：
		<fmt:setTimeZone value="时区值" var="varName" scope="page/session/request/application" />

**5)parseNumber标签：**
1. parseNumber标签用于将以字符串表示的数字、货币或者百分比解析成数字。
和formatNumber的作用相反，但是用法相似。
2. parseNumber有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-17.jpg)
3. 使用方法：
无body content:
`<fmt:parseNumber value="源值" [各项属性配置] />`
有body content:
		<fmt:parseNumber [各项属性配置]>
			要被转换的数字值
		</fmt:parseNumber>
各属性和可选值如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-18.jpg)

**6)parseDate标签：**
1. parseDate标签用于将时间表时的格式解析为字符串日期和时间。
和formatDate标签相反，用法相似。
2. 有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-19.jpg)
3. 有两种使用方式： 
无body content:
		<fmt:parseDate value="源值" [各项属性配置] />
有body content:
		<fmt:parseDate [各项属性配置]>
			要被转换的值
		</fmt:parseNumber>
4. 个属性可选值如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-20.jpg)

---
## 8.SQL相关的行为(标签)
**1)简介：**
1. 需要导入依赖：
`<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>`
2. 有如下标签：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-21.jpg)
3. 参考：
<http://www.runoob.com/jsp/jsp-jstl.html>

**2)setDataSource标签：**
1. 用来配置数据源或者将数据源信息存储在某作用域的变量中，用来作为其它JSTL数据库操作的数据源。
2. 有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-22.jpg)
3. 使用方法：
		<sql:setDataSource
		  var="<string>"
		  scope="<string>"
		  dataSource="<string>"
		  driver="<string>"
		  url="<string>"
		  user="<string>"
		  password="<string>"/>
4. 简单示例：
		<sql:setDataSource var="snapshot" 
			driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost/TEST"
			user="user_id" password="password"/>
		<sql:query dataSource="${snapshot}" sql="..." var="result" />

**3)query标签：**
1. 用于运行sql查询语句，将结果存储在有界变量中。
2. 有以下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-23.jpg)
3. 使用方式如下：
		<sql:query
		  var="<string>"
		  scope="<string>"
		  sql="<string>"
		  dataSource="<string>"
		  startRow="<string>"
		  maxRows="<string>"/>
4. 使用示例：
		<sql:setDataSource var="snapshot" driver="com.mysql.jdbc.Driver"
			url="jdbc:mysql://localhost/TEST"
			user="root"  password="pass123"/>
		
		<sql:query dataSource="${snapshot}" var="result">
		SELECT * from Employees;
		</sql:query>
		 
		<table border="1" width="100%">
		<tr>
		<th>Emp ID</th>
		<th>First Name</th>
		<th>Last Name</th>
		<th>Age</th>
		</tr>
		<c:forEach var="row" items="${result.rows}">
		<tr>
		<td><c:out value="${row.id}"/></td>
		<td><c:out value="${row.first}"/></td>
		<td><c:out value="${row.last}"/></td>
		<td><c:out value="${row.age}"/></td>
		</tr>
		</c:forEach>
		</table>

**4)update标签：**
1. 用来执行一个没有返回值的SQL语句，比如SQL INSERT，UPDATE，DELETE语句。
2. 有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-24.jpg)
3. 使用方式：
		<sql:update var="<string>" scope="<string>" sql="<string>" dataSource="<string>"/>

**5)param标签：**
1. 与`<sql:query>`标签和`<sql:update>`标签嵌套使用，用来提供一个值占位符。如果是一个null值，则将占位符设为SQLNULL。
2. 只有一个属性value，表示需要设置的参数值。
3. 使用示例：
		<sql:update dataSource="${snapshot}" var="count">
		  DELETE FROM Employees WHERE Id = ?
		  <sql:param value="${empId}" />
		</sql:update>

**6)dateParam标签：**
1. 与`<sql:query>`标签和`<sql:update>`标签嵌套使用，用来提供日期和时间的占位符。如果是一个null值，则将占位符设为SQLNULL。
2. 有两个属性：
	- value：表示需要设置的日期参数（java.util.Date）
	- type：类型，DATE（只有日期），TIME（只有时间），TIMESTAMP（日期和时间）
3. 使用示例：
		<sql:update dataSource="${snapshot}" var="count">
		   UPDATE Students SET dob = ? WHERE Id = ?
		   <sql:dateParam value="<%=DoB%>" type="DATE" />
		   <sql:param value="<%=studentId%>" />
		</sql:update>

**7)transaction标签：**
1. 用来将`<sql:query>`标签和`<sql:update>`标签封装至事务中。可以将大量的`<sql:query>`和`<sql:update>`操作装入`<sql:transaction>`中，使它们成为单一的事务。
2. 有如下属性：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-25.jpg)
3. 使用示例：
		<sql:transaction dataSource="${snapshot}">
		   <sql:update var="count">
		      UPDATE Students SET last = 'Ali' WHERE Id = 102
		   </sql:update>
		   <sql:update var="count">
		      UPDATE Students SET last = 'Shah' WHERE Id = 103
		   </sql:update>
		   <sql:update var="count">
		     INSERT INTO Students 
		     VALUES (104,'Nuha', 'Ali', '2010/05/26');
		   </sql:update>
		</sql:transaction>

---
## 9.JSTL相关函数
**1)函数简介：**
1. JSTL包含一系列标准函数，大部分是通用的字符串处理函数。(类似java的String的函数)
引用JSTL函数库的语法如下：
		<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
2. 函数一览：
![](http://p5ki4lhmo.bkt.clouddn.com/00067SpringMVC%E5%AD%A6%E4%B9%A05-26.jpg)
3. 下面对这些函数进行详细介绍

### contains~indexof函数
**1)contains()函数：**
1. 测试一个字符串是否包含指定的子字符串。
2. 语法：
`${fn:contains(string,substring)}`
string可以为静态字符串也可以为变量。
可以配合`<c:if>`标签使用：
		<c:if test="${fn:contains(<原始字符串>, <要查找的子字符串>)}">
		...
		</c:if> 

**2)containsIgnoreCase()函数：**
1. 用于确定一个字符串是否包含指定的子串，忽略大小写。
（和contains差不多，只是忽略大小写）
2. 使用方式和contains类似：
		<c:set var="theString" value="I am from runoob"/>
		<c:if test="${fn:containsIgnoreCase(theString, 'runoob')}">
			<p>找到 runoob<p>
		</c:if>
		<c:if test="${fn:containsIgnoreCase(theString, 'RUNOOB')}">
			<p>找到 RUNOOB<p>
		</c:if>

**3)endsWith()函数：**
1. 判断一个字符串是否以指定后缀结尾。
2. 使用方式：
		<c:if test="${fn:endsWith(<原始字符串>, <要查找的子字符串>)}">
		...
		</c:if>
3. 使用示例：
		<c:set var="theString" value="I am from runoob 123"/>
		<c:if test="${fn:endsWith(theString, '123')}">
		   <p>字符串以 123 结尾<p>
		</c:if>
		<c:if test="${fn:endsWith(theString, 'unoob')}">
		   <p>字符串以 runoob 结尾<p>
		</c:if>

**4)escapeXml()函数：**
1. fn:escapeXml()函数忽略用于XML标记的字符。
2. 使用方式：
		${fn:escapeXml(<要转义标记的文本>)} 

**5)indexOf()函数：**
1. fn:indexOf()函数返回一个字符串中指定子串的位置。
2. 使用方式：
		${fn:indexOf(<原始字符串>,<子字符串>)}
3. 使用示例：
		<c:set var="string1" value="This is first String."/>
		<c:set var="string2" value="This <abc>is second String.</abc>"/>
		<p>Index (1) : ${fn:indexOf(string1, "first")}</p>
		<p>Index (2) : ${fn:indexOf(string2, "second")}</p>

### join~startsWith函数
**1)join()函数：**
1. fn:join()函数将一个数组中的所有元素使用指定的分隔符来连接成一个字符串。
2. 使用方式：
		${fn:join([数组], <分隔符>)}
3. 使用示例：(输出 www-runoob-com)
		<c:set var="string1" value="www runoob com"/>
		<c:set var="string2" value="${fn:split(string1, ' ')}" />
		<c:set var="string3" value="${fn:join(string2, '-')}" />
		<p>字符串为 : ${string3}</p>

**2)length()函数：**
1. fn:length()函数返回字符串长度或集合中元素的数量。
2. 使用方式：
		${fn:length(collection | string)}
3. 使用示例：(结果21，22)
		<c:set var="string1" value="This is first String."/>
		<c:set var="string2" value="This is second String." />
		<p>字符串长度 (1) : ${fn:length(string1)}</p>
		<p>字符串长度 (2) : ${fn:length(string2)}</p>

**3)replace()函数：**
1. fn:replace()函数将字符串中所有指定的子串用另外的字符串替换。
2. 使用方式：
		${fn:replace(<原始字符串>, <被替换的字符串>, <要替换的字符串>)}
3. 使用示例：
		<c:set var="string1" value="I am from google"/>
		<c:set var="string2" value="${fn:replace(string1, 'google', 'runoob')}" />

**4）split()函数：**
1. fn:split()函数将一个字符串用指定的分隔符分裂为一个子串数组。
2. 使用方式：
		${fn:split(<带分隔符的字符串>, <分隔符>)}
3. 使用示例：
		<c:set var="string1" value="www runoob com"/>
		<c:set var="string2" value="${fn:split(string1, ' ')}" />
		<c:set var="string3" value="${fn:join(string2, '-')}" />

**5)startsWith()函数：**
1. fn:startsWith()函数用于确定一个字符串是否以指定的前缀开始。
(和endsWith()函数相反)
2. 使用方式：
		<c:if test="${fn:startsWith(<原始字符串>, <搜索的前缀>)}">
			...
		</c:if>
3. 使用示例：
		<c:set var="string" value="Runoob: I am from Runoob."/>
		<c:if test="${fn:startsWith(string, 'Google')}"> 
			<p>字符串以 Google 开头</p>
		</c:if>

### substring~trim函数
**1)substring()函数：**
1. fn:substring()函数返回字符串中指定开始和结束索引的子串。
2. 使用方式：
		${fn:substring(<string>, <beginIndex>, <endIndex>)}
3. 使用示例：
		<c:set var="string1" value="This is first String."/>
		<c:set var="string2" value="${fn:substring(string1, 5, 15)}" />

**2)substringAfter()/substringBefore()函数：**
1. fn:substringAfter()函数返回字符串中指定子串后面的部分。
fn:substringBefore()函数返回一个字符串中指定子串前面的部分。
2. 使用方式：
		${fn:substringAfter(<string>, <substring>)}
		${fn:substringBefore(<string>, <substring>)}
3. 使用示例：
		<c:set var="string1" value="This is first String."/>
		<c:set var="string2" value="${fn:substringAfter(string1, 'is')}" />
		<c:set var="string3" value="${fn:substringBefore(string1, 'is')}" />

**3)toLowerCase()/toUpperCase()函数：**
1. fn:toUpperCase()函数将一个字符串中的所有字符转为大写。
fn:toLowerCase()函数将字符串中的所有字符转为小写。
2. 使用方式：
		${fn.toUpperCase(<string>)}
		${fn.toLowerCase(<string>)}
3. 使用示例：
		<c:set var="string1" value="I am from runoob"/>
		<c:set var="string2" value="${fn:toUpperCase(string1)}" />
		<c:set var="string3" value="${fn:toLowerCase(string2)}" />

**4)trim()函数：**
1. fn:trim()函数将字符串两端的空白符移除。
2. 使用方式：
		${fn:trim(<string>)}
3. 使用示例：
		<c:set var="string1" value="I am from runoob         "/>
		<p>string1 长度 : ${fn:length(string1)}</p>
		
		<c:set var="string2" value="${fn:trim(string1)}" />
		<p>string2 长度 : ${fn:length(string2)}</p>
		<p>字符串为 : ${string2}</p>
测试结果：
		string1 长度 : 25
		string2 长度 : 16
		字符串为 : I am from runoob

---
