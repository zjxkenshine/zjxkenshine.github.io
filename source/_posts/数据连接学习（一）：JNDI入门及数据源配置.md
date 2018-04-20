---
title: 数据连接学习（一）：JNDI入门及数据源配置
date: 2018-4-19 21:10:26 
tags:
- Java
- JNDI
categories: 数据库

---
## 0.学习原因
因为学习Spring时涉及到了很多关于JNDI的知识，卧槽，压根没听说过呀，所以特地来学习一下。
参考博客：
[JNDI学习总结(一)——JNDI数据源的配置](http://www.cnblogs.com/xdp-gacl/p/3951952.html)

---
## 1.JNDI的诞生及简介简介
**1)服务器数据源配置的诞生**
1. JDBC阶段：
一开始是使用JDBC来连接操作数据库的：
在Java开发中，使用JDBC操作数据库的四个步骤如下：
	>①加载数据库驱动程序(Class.forName("数据库驱动类");)
	②连接数据库(Connection con  = DriverManager.getConnection();)
	③操作数据库(PreparedStatement stat = con.prepareStatement(sql);stat.executeQuery();)
	④关闭数据库，释放连接(con.close();)
2. 所有的用户都需要经过此四步进行操作，但是这四步之中有三步(①加载数据库驱动程序、②连接数据库、④关闭数据库，释放连接)对所有人都是一样的，而所有人只有在操作数据库上是不一样，那么这就造成了性能的损耗。
那么最好的做法是，准备出一个空间，此空间里专门保存着全部的数据库连接，以后用户用数据库操作的时候不用再重新加载驱动、连接数据库之类的，而直接从此空间中取走连接，关闭的时候直接把连接放回到此空间之中。
而这个空间就是**连接池**。
3. 但是连接池的是有会有以下疑虑：
	>1、 如果没有任何一个用户使用连接，那么那么应该维持一定数量的连接，等待用户使用。
	2、 如果连接已经满了，则必须打开新的连接，供更多用户使用。
	3、 如果一个服务器就只能有100个连接，那么如果有第101个人过来呢？应该等待其他用户释放连接
	4、 如果一个用户等待时间太长了，则应该告诉用户，操作是失败的。
4. 如果直接用程序实现以上功能，则会比较麻烦。
所以在Tomcat 4.1.27之后，在服务器上就直接增加了数据源的配置选项，**直接在服务器上配置好数据源连接池即可**。在J2EE服务器上保存着一个数据库的多个连接。每一个连接通过DataSource可以找到。DataSource被绑定在了JNDI树上（为每一个DataSource提供一个名字）客户端通过名称找到在JNDI树上绑定的DataSource，再由DataSource找到一个连接。
5. 示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-01.jpg)
6. 网上流传的图：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-02.jpg)
7. 那么在以后的操作中，除了数据库的连接方式不一样之外，其他的所有操作都一样，只是关闭的时候不是彻底地关闭数据库，而是把数据库的连接放回到连接池中去。

**2)JNDI的简介及优点：**
1. JNDI的定义：
上面知道了JNDI的作用，JNDI的定义也就很容易理解了：
JNDI是Java命名与文件夹接口（Java Naming and Directory Interface），在J2EE规范中是重要的规范之中的一个。
简单的理解，区别于JDBC,就是：
jdbc是java去找数据库驱动，jndi是通过你的服务器配置（如Tomcat）的配置文件context来找数据库驱动~
2. 现在JNDI已经成为J2EE的标准之一，所有的J2EE容器都必须提供一个JNDI的服务。(web容器就是Tomcat等服务器充当的)
3. 为什么使用了连接池还要用JNDI：
	>为了数据库资源的管理，在容器中配置一个数据库连接池，使用JNDI 来管理
	这样容器中运行多个服务的时候，每个服务只需添加一个jndi的名称就可以连接到数据库了
	如果不使用jndi的方式，直接在项目中配置数据库连接池，那么每个项目需要配置一次，如果更改数据库地址时，
	每个项目的数据库连接方式都要更改，比较麻烦，使用jndi的话，直接更改一下jndi里面的数据库连接池的配置就可以了，方便一些。
4. 一般来说如果目标客户有专业的应用服务器，比如 WebSphere，WebLogic，我们就不需要在代码中配置使用特定的 dbcp 或其它的连接池了。只使用JNDI就可以了。

---
## 2.JNDI+Tomcat配置数据源(全局配置)
**1)Tomcat全局配置：**
1. 需要在tomcat中的server.xml中配置数据源：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-03.jpg)
2. 在Tomcat的lib目录下添加驱动包：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-04.jpg)
3. 打开server.xml，一般都有一个默认的全局配置(Resource)标签：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-05.jpg)
代码如下：
		<GlobalNamingResources>
			<Resource auth="Container" description="User databasethat can be updated and saved" 
			factory="org.apache.catalina.users.MemoryUserDatabaseFactory" 
			name="UserDatabase" 
			pathname="conf/tomcat-users.xml" 
			type="org.apache.catalina.UserDatabase"/>
		</GlobalNamingResources>
4. 在默认的Resource下添加配置如下：
		<Resource 
			name="jdbc/mysql"
			auth="Container" 
			type="javax.sql.DataSource"
			maxActive="100" 
			maxIdle="30" 
			maxWait="10000"
			username="root" 
			password="root"
			driverClassName="com.mysql.jdbc.Driver"
			url="jdbc:mysql://127.0.0.1:3306/springtest?useUnicode=true&amp;characterEncoding=utf-8"/>
各项配置的含义后面介绍。
5. 以上两步(导包+配置server.xml)全局配置就已经配置好了。
**这一步并非必须的，可以不写，亲测没有失败**
在需要使用的JNDI的项目中的web.xml中引用该配置：
		<resource-ref>
			<res-ref-name>jdbc/mysql</res-ref-name>
			<res-type>javax.sql.DataSource</res-type>
			<res-auth>Container</res-auth>
		</resource-ref>
属于可选配置，如果在web.xml中加入了上面的配置，则需要在Tomcat中一定要配置对应的Resource，否则会报错。
6. 查看Service的最下面，可以看到有当前工程的上下文：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-06.jpg)
在该上下文中增加对全局配置的引用：
		<Context docBase="JNDITest" path="/JNDITest" reloadable="true" source="org.eclipse.jst.jee.server:JNDITest">
				<ResourceLink global="mysqlData" name="jdbc/mysql" type="javax.sql.DataSource" />
		</Context>
配置模板：
		<?xml version="1.0" encoding="UTF-8"?>
		<!--
		jndi配置方法（tomcat）：
		 -->
		<!--映射JNDITest项目的虚拟目录-->
		<Context docBase="D:/MyEclipse8.5/workspace/JNDITest/WebRoot" debug="0" reloadable="false">
			<!--global指的是全局配置的name,
				name指的是这个DataSourse的名字-->
			<!--引用全局配置-->
			<ResourceLink name="mysqlData" global="jdbc/mysql" type="javax.sql.DataSource"/>
		</Context>
7. 这一步可以替换第6步（写了这个则不用配置第六步）：
在项目的META-INF下面建立context.xml文件，在里面写上：
		<?xml version="1.0" encoding="UTF-8"?> 
		<Context> 
			<ResourceLink name="mysqlData" global="jdbc/mysql type="javax.sql.DataSource"/> 
		</Context>
8. 第6步替换策略二（自认为最简单的方式）：
找到Tomcat下的conf/context.xml文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-08.jpg)
配置上：
		<Context>
			<WatchedResource>WEB-INF/web.xml</WatchedResource>
			<WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
			<ResourceLink name="mysqlData" global="jdbc/mysql" type="javax.sql.DataSource"/>
		</Context>
这句话就可以了。
所以第六步可以有三种策略(我只测试了一种)。

**2)创建测试类代码：**
1. 创建测试类：(最简单的jsp测试)
		<%@page import="javax.naming.InitialContext"%>  
		<%@page import="javax.naming.Context"%>  
		<%@page import="javax.sql.DataSource"%>  
		<%@ page language="java" contentType="text/html; charset=utf-8"  
			pageEncoding="utf-8"%>  
		<%  
			Context initContext = new InitialContext();		//上转型，也可以不用上转型 
			DataSource ds = (DataSource)initContext.lookup("java:/comp/env/mysqlData");  
			out.print(ds); 
		%> 
2. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-07.jpg)
成功获取到数据源(javax.sql.datasource对象)。
3. 注意事项：
`java:/comp/env/`，这是j2ee的命名空间，其他地方可能会变
`context.lookup(“XXX”)`，在任何时候都是有效的，只要XXX确实是一个存在的JNDI名。
Tomcat的全局JNDI资源不能直接访问，必须有java:comp/env/前缀。
这样子配置好的工程默认使用的是Tomcat自带的dbcp连接池。

**3)JNDI的配置模板：**

	<GlobalNamingResources>
		<!--默认的这个配置不要删除-->
		<Resource name="UserDatabase" auth="Container"
				type="org.apache.catalina.UserDatabase"
				description="User database that can be updated and saved"
				factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
				pathname="conf/tomcat-users.xml" />
		
		<!--
		|- name：表示以后要查找的名称。通过此名称可以找到DataSource，此名称任意更换，但是程序中最终要查找的就是此名称，
		为了不与其他的名称混淆，所以使用jdbc/oracle，现在配置的是一个jdbc的关于oracle的命名服务。
		|- auth：由容器进行授权及管理，指的用户名和密码是否可以在容器上生效
		|- type：此名称所代表的类型，现在为javax.sql.DataSource
		|- maxActive：表示一个数据库在此服务器上所能打开的最大连接数
		|- maxIdle：表示一个数据库在此服务器上维持的最小连接数
		|- maxWait：最大等待时间。10000毫秒
		|- username：数据库连接的用户名
		|- password：数据库连接的密码
		|- driverClassName：数据库连接的驱动程序
		|- url：数据库连接的地址
		-->
		
		<!--配置Oracle数据库的JNDI数据源-->
		<Resource 
				name="jdbc/oracle"
				auth="Container" 
				type="javax.sql.DataSource"
				maxActive="100" 
				maxIdle="30" 
				maxWait="10000"
				username="lead_oams" 
				password="p"
				driverClassName="oracle.jdbc.driver.OracleDriver"
				url="jdbc:oracle:thin:@127.0.0.1:1521:lead"/>
		
		<!--配置MySQL数据库的JNDI数据源-->
		<Resource 
				name="jdbc/mysql"
				auth="Container" 
				type="javax.sql.DataSource"
				maxActive="100" 
				maxIdle="30" 
				maxWait="10000"
				username="root" 
				password="root"
				driverClassName="com.mysql.jdbc.Driver"
				url="jdbc:mysql://127.0.0.1:3306/leadtest?useUnicode=true&amp;characterEncoding=utf-8"/>
		
		<!--配置SQLServer数据库的JNDI数据源-->
		<Resource 
				name="jdbc/sqlserver"
				auth="Container" 
				type="javax.sql.DataSource"
				maxActive="100" 
				maxIdle="30" 
				maxWait="10000"
				username="sa" 
				password="p@ssw0rd"
				driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
				url="jdbc:sqlserver://127.0.0.1:1433;DatabaseName=demo"/>
		
	</GlobalNamingResources>

---
## 3.JNDI+Tomcat配置数据源(局部配置)
**1)简单思路**
1. 非全局JNDI数据源是针对某一个Web项目配置的数据源，具体的配置步骤如下：
1、在tomcat服务器的lib目录下加入数据库连接的驱动jar包
2、针对具体的web项目映射虚拟目录，然后在虚拟目录映射的配置文件中配置JNDI数据源
2. 还有一个更简单的思路：
就是将上述引用全局配置的地方改为和全局配置相同的那样的配置,并将全局配置删除。
所以共有三种方法(conf/context.xml中配置的可能是全局的)
3. 正常使用的话还是用全局配置吧。

**2)配置过程及几种方法：**
配置模板和全局配置的相同。
1. 方法一：修改server.xml中的context(对应全局配置步骤6)
		<Context docBase="JNDITest" path="/JNDITest" reloadable="true" source="org.eclipse.jst.jee.server:JNDITest">
			<Resource name="jdbc/test" 
					auth="Container" 
					type="javax.sql.DataSource" 
					driverClassName="com.mysql.jdbc.Driver" 
					url="jdbc:mysql://127.0.0.1/test" 
					username="root" 
					password="root" 
					maxActive="20" 
					maxIdle="10" 
					maxWait="-1"/> 
		</Context>
2. 方法二：在项目的META-INF下面建立context.xml文件（对应全局配置步骤7)，在里面配置上：
		<?xml version="1.0" encoding="UTF-8"?> 
		<Context> 
			<Resource name="jdbc/test" 
					auth="Container" 
					type="javax.sql.DataSource" 
					driverClassName="com.mysql.jdbc.Driver" 
					url="jdbc:mysql://127.0.0.1/test" 
					username="root" 
					password="root" 
					maxActive="20" 
					maxIdle="10" 
					maxWait="-1"/> 
		</Context> 
3. 方法三：配置conf/context.xml（对应步骤8）
(这步极有可能是全局的配置)：
		<Context>
			<WatchedResource>WEB-INF/web.xml</WatchedResource>
			<WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
			<Resource name="jdbc/test" 
					auth="Container" 
					type="javax.sql.DataSource" 
					driverClassName="com.mysql.jdbc.Driver" 
					url="jdbc:mysql://127.0.0.1/test" 
					username="root" 
					password="root" 
					maxActive="20" 
					maxIdle="10" 
					maxWait="-1"/> 
		<\Context>
将上述WatchedResource标签中的路径改为项目路径就是局部配置了。
4. 测试类和全局配置的相同，我也不测试了。

---
## 4.关于配置JNDI的总结
1. 全局配置的思路图(虚线为非必须)：
![](http://p5ki4lhmo.bkt.clouddn.com/00042JNDI%E5%AD%A6%E4%B9%A01-09.jpg)
2. 局部配置就不画了，差不多。
3. 以上步骤就能配置数据源了，至于获取了DataSource怎么操作数据，那就是javax.sql.DataSource的事情了,以后再学。

---
