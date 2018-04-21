---
title: 数据连接学习（二）：DBCP连接池的简单使用
date: 2018-4-20 10:10:26 
tags:
- Java
- 连接池
- Commons
categories: 数据库

---
## 0.学习原因
学习Spring时涉及到了这方面的知识，就来学一下。
以前学过，但是忘的差不多了。

学习博客：
<http://blog.csdn.net/shuaihj/article/details/14223015>
<https://www.cnblogs.com/sunseine/p/5947448.html>
<http://happyqing.iteye.com/blog/2304131>

---
## 1.连接池简介
1. 简单来说：
就是一个创建和管理数据库连接的缓冲池。类似缓存，已经存好了多个连接，使用时直接取出使用，使用完毕再放回连接池，不用关闭连接。
2. 对于大多数应用程序，当它们正在处理通常需要数毫秒完成的事务时，仅需要能够访问JDBC连接的 1 个线程。当不处理事务时，这个连接就会闲置。相反，连接池允许闲置的连接被其它需要的线程使用。
3. 一般的JDBC连接的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-01.jpg)
如果有多个人请求，则需要多个连接的开启与关闭。
4. 使用连接池的操作：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-02.jpg)
连接可以共用，不用为每个请求额外创建一个JDBC连接。
5. 常用的Java连接池有两个：DBCP和C3P0。
而DBCP又分为了dbcp与dbcp2。

---
## 2.连接池的简单实现
**1）原理与简单实现：**
1. 一个最简单的实现：
		public class MyDataSource implements DataSource {
			//链表 --- 实现栈结构
			private LinkedList<Connection> dataSources = new LinkedList<Connection>();
			//初始化连接数量
			public MyDataSource() {
				//一次性创建10个连接
				for(int i = 0; i < 10; i++) {
					try {
						//1、装载sqlserver驱动对象
						DriverManager.registerDriver(new SQLServerDriver());
						//2、通过JDBC建立MySql数据库连接
						Connection con =DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/sqltest?useUnicode=true&characterEncoding=UTF-8", "root", "root");
						//3、将连接加入连接池中
						dataSources.add(con);
					 } catch (Exception e) {
						e.printStackTrace();
					 }
				}
			}
			@Override
			public Connection getConnection() throws SQLException {
				//取出连接池中一个连接
				finalConnection conn = dataSources.removeFirst(); // 删除第一个连接返回
				return conn;
			}
			//将连接放回连接池
			public void releaseConnection(Connection conn) {
				dataSources.add(conn);
			}
		}
2. 连接池大大提供了数据库连接的利用率，减小了内存吞吐的开销。
但是连接池还需要考虑许多问题。

**2)连接池还需要考虑的问题：**
1. 并发问题：
为了使连接管理服务具有最大的通用性，必须考虑多线程环境，即并发问题。
可以使用Synchronized关键字解决(更多解决方案参考多线程及并发的博客)。
2. 多数据库服务器：
 对于大型的企业级应用，常常需要同时连接不同的数据库（如连接oracle和sybase）。如何连接不同的数据库呢？
解决方案：
	>设计一个符合单例模式的连接池管理类，在连接池管理类的唯一实例被创建时读取一个资源文件，其中资源文件中存放着多个数据库的url地址等信息。根据资源文件提供的信息，创建多个连接池类的实例，每一个实例都是一个特定数据库的连接池。连接池管理类实例为每个连接池实例取一个名字，通过不同的名字来管理不同的连接池。
3. 多用户环境：
对于同一个数据库有多个用户使用不同的名称和密码访问的情况，也可以通过资源文件处理。
4. 事务处理：
在java语言中，connection类本身提供了对事务的支持，可以通过设置connection的autocommit属性为false 然后显式的调用commit或rollback方法来实现。但要高效的进行connection复用，就必须提供相应的事务支持机制。可采用每一个事务独占一个连接来实现，这种方法可以大大降低事务管理的复杂性。
5. 连接池的分配与释放
连接池的分配与释放，对系统的性能有很大的影响。合理的分配与释放，可以提高连接的复用度，从而降低建立新连接的开销，同时还可以加快用户的访问速度。
解决方案示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-03-1.jpg)
解决方案描述：
>对于连接的管理可使用空闲池。即把已经创建但尚未分配出去的连接按创建时间存放到一个空闲池中。每当用户请求一个连接时，系统首先检查空闲池内有没有空闲连接。如果有就把建立时间最长（通过容器的顺序存放实现）的那个连接分配给他（实际是先做连接是否有效的判断，如果可用就分配给用户，如不可用就把这个连接从空闲池删掉，重新检测空闲池是否还有连接）；如果没有则检查当前所开连接池是否达到连接池所允许的最大连接数（maxconn）如果没有达到，就新建一个连接，如果已经达到，就等待一定的时间（timeout）。如果在等待的时间内有连接被释放出来就可以把这个连接分配给等待的用户，如果等待时间超过预定时间timeout 则返回空值（null）。系统对已经分配出去正在使用的连接只做计数，当使用完后再返还给空闲池。对于空闲连接的状态，可开辟专门的线程定时检测，这样会花费一定的系统开销，但可以保证较快的响应速度。也可采取不开辟专门线程，只是在分配前检测的方法。
6. 连接池的配置与维护：
最大连接数是连接池中允许连接的最大数目，具体设置多少，要看系统的访问量，可通过反复测试，找到最佳点。
如何确保连接池中的最小连接数呢？有动态和静态两种策略。动态即每隔一定时间就对连接池进行检测，如果发现连接数量小于最小连接数，则补充相应数量的新连接以保证连接池的正常运转。静态是发现空闲连接不够时再去检查。

**3)开发中最好使用成熟的连接池：**
1. 已经存在很多流行的性能优良的第三方数据库连接池jar包供我们使用
2. DBCP:
<http://commons.apache.org/proper/commons-dbcp/>
3. C3P0: 
<http://mvnrepository.com/artifact/com.mchange/c3p0>

---
## 3.DBCP的简介及环境搭建
**1)DBCP简介：**
1. DBCP(DataBase connection pool)数据库连接池是 apache 上的一个Java连接池项目。DBCP通过连接池预先同数据库建立一些连接放在内存中(即连接池中)，应用程序需要建立数据库连接时直接到从接池中申请一个连接使用，用完后由连接池回收该连接，从而达到连接复用，减少资源消耗的目的。
2. 目前的DBCP已经出到1.4版本。
而dbcp2与dbcp并不是完全相同的，用法也不同。
dbcp2已经出到2.2.0版本。
dbcp2需要jdk1.7版本的支持，否则会报错。

**2)测试环境搭建（dbcp2）：**
1. 非Maven环境下只需要创建项目并且导入jar包就可以了。
要导入的jar包(dbcp2）：
commons-dbcp2.jar
commons-pool2.jar
dbcp则导入：
commons-dbcp.jar
commons-pool.jar
还看见有博客说要导入：
commons-logging.jar
commons-collections.jar
那就都加上呗，反正多了无害,等以后有空了再来细细研究这些jar包的作用。
2. Maven环境下，我就只创建简单的java工程来测试了，pom.xml依赖如下：
		...
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>  
			<groupId>org.apache.commons</groupId>  
			<artifactId>commons-dbcp2</artifactId>  
			<version>2.2.0</version>  
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>commons-collections</groupId>
			<artifactId>commons-collections</artifactId>
			<version>3.2.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-pool2</artifactId>
			<version>2.5.0</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
		...
这样就搭建完毕了。

---
## 4.简单使用dbcp2
1. 在src/main/resources下创建dbcp.properties文件
2. dbcp.properties文件配置如下：
		######## DBCP配置文件 ##########
		# 驱动名
		driverClassName=com.mysql.jdbc.Driver
		# url
		url=jdbc:mysql://127.0.0.1:3306/springtest01?useUnicode=true&characterEncoding=utf-8
		# 用户名
		username=root
		# 密码
		password=root
		# 初始连接数
		initialSize=30
		# 最大活跃数
		maxTotal=30
		# 最大空闲数
		maxIdle=10
		# 最小空闲数
		minIdle=5
		# 最长等待时间(毫秒)
		maxWaitMillis=1000
		# 程序中的连接不使用后是否被连接池回收(该版本要使用removeAbandonedOnMaintenance和removeAbandonedOnBorrow)
		# removeAbandoned=true
		removeAbandonedOnMaintenance=true
		removeAbandonedOnBorrow=true
		# 连接在所指定的秒数内未使用才会被删除(秒)
		removeAbandonedTimeout=100
3. 创建工具类DBCPUtil：
		package DBCP2;
		import java.io.FileInputStream;
		import java.io.IOException;
		import java.sql.Connection;
		import java.sql.SQLException;
		import java.util.Properties;
		import javax.sql.DataSource;
		import org.apache.commons.dbcp2.BasicDataSourceFactory;
		public class DBCPUtil {
			private static Properties properties = new Properties();
			private static DataSource dataSource;
			//加载DBCP配置文件
			static{
			   try{
				  //注意这里需要使用绝对路径
			      FileInputStream is = new FileInputStream("src/main/resources/dbcp.properties");  
			      properties.load(is);
			   }catch(IOException e){
			      e.printStackTrace();
			   }
			   //获取数据源对象
			   try{
				   dataSource = BasicDataSourceFactory.createDataSource(properties);
			   }catch(Exception e){
			       e.printStackTrace();
			   }
			}
			//从连接池中获取一个连接
			public static Connection getConnection(){
			   Connection connection = null;
			   try{
		            connection = dataSource.getConnection();
		        }catch(SQLException e){
		            e.printStackTrace();
		        }
		        try {
		            connection.setAutoCommit(false);
		        } catch (SQLException e) {
		            e.printStackTrace();
		        }
		        return connection;
		    }
		}
4. 测试类：
		package DBCP2;
		
		import java.sql.Connection;
		import org.junit.Assert;
		import org.junit.Test;
		import DBCP2.DBCPUtil;
		public class DBCPTest {
			@Test
			public void testConn(){
				Connection conn=DBCPUtil.getConnection();
				Assert.assertNotNull(conn);
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-05.jpg)

---
## 5.dbcp2的另一种使用方式
1. 其实还有很多种方式，只要能够加载配置就可以了。
2. 创建DBCPUtil2如下：
		package DBCP2;
		import java.sql.Connection;
		import java.sql.SQLException;
		import org.apache.commons.dbcp2.BasicDataSource;
		public class DBCPUtil2{
			private static BasicDataSource dataSource=new BasicDataSource();
			//加载DBCP配置文件
			static{
			   //获取数据源对象
			   try{
				   dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/springtest?useUnicode=true&characterEncoding=utf-8");
				   dataSource.setUsername("root");
				   dataSource.setPassword("root");
				   dataSource.setDriverClassName("com.mysql.jdbc.Driver");
				   //初始连接数
				   dataSource.setMaxTotal(30);
				   //最大空闲数
				   dataSource.setMaxIdle(10);
				   //最小空闲数
				   dataSource.setMinIdle(5);
				   //最长等待时间(ms)
				   dataSource.setMaxWaitMillis(10000);
				   //指定时间内未使用连接则关闭(s)
				   dataSource.setRemoveAbandonedTimeout(100);
				   //程序中的连接不使用后是否被连接池回收
				   dataSource.setRemoveAbandonedOnBorrow(true);
				   dataSource.setRemoveAbandonedOnMaintenance(true);
			   }catch(Exception e){
			       e.printStackTrace();
			   }
			}
			//从连接池中获取一个连接
			public static Connection getConnection(){
			   Connection connection = null;
			   try{
		            connection = dataSource.getConnection();
		        }catch(SQLException e){
		            e.printStackTrace();
		        }
		        try {
		            connection.setAutoCommit(false);
		        } catch (SQLException e) {
		            e.printStackTrace();
		        }
		        return connection;
		    }
		}
3. 测试代码：
		package DBCP2;
		import java.sql.Connection;
		import org.junit.Assert;
		import org.junit.Test;
		public class DBCPTest2 {
			@Test
			public void testConn(){
				Connection conn=DBCPUtil2.getConnection();
				System.out.println(conn);
				Assert.assertNotNull(conn);
			}
		}
4. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-06.jpg)
5. 其他使用方式自行探索
其实JNDI也算一种，因为在Tomcat中内置了DBCP2连接池。
6. 获取连接之后的事情，就是和JDBC的实现方式相同了。
7. 总结这两种种方式的思维图：
![](http://p5ki4lhmo.bkt.clouddn.com/00043%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E5%AD%A6%E4%B9%A02-07.jpg)

---
## 6.DBCP连接池在Spring中的基本配置
参考博客：
<http://happyqing.iteye.com/blog/2304131>
### Spring使用dbcp
1. 首先需要导包：
		<dependency>  
			<groupId>commons-dbcp</groupId>  
			<artifactId>commons-dbcp</artifactId>  
			<version>1.4</version>  
		</dependency> 
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-pool2</artifactId>
			<version>2.5.0</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
2. xml文件：
		<!-- 属性文件配置 -->  
		<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
		    <property name="locations">  
		        <list>  
		            <value>classpath:jdbc.properties</value>  
		        </list>  
		    </property>  
		</bean>  
		  
		<!-- dbcp -->  
		<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
		    destroy-method="close">  
		    <property name="driverClassName" value="${driverClassName}" />  
		    <property name="url" value="${url}" />  
		    <property name="username" value="${username}" />  
		    <property name="password" value="${password}" />  
		    <!-- 连接初始值，连接池启动时创建的连接数量的初始值  默认值是0 -->  
		    <property name="initialSize" value="3" />  
		    <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请  默认值是0 -->  
		    <property name="minIdle" value="3" />  
		    <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 ，0时无限制  默认值是8 -->  
		    <property name="maxIdle" value="5" />  
		    <!-- 连接池的最大值，同一时间可以从池分配的最多连接数量，0时无限制   默认值是8 -->  
		    <property name="maxActive" value="15" />  
		</bean>
3. jdbc.properties:
		# dbcp  
		driverClassName=com.mysql.jdbc.Driver  
		url=jdbc:mysql://localhost:3306/springtest01?useUnicode=true&characterEncoding=utf-8  
		username=root  
		password=password  

### Spring使用dbcp2
**1)xml配置方式：**
1. 导包
2. xml配置:
		<!-- dbcp2 -->  
		<bean id="dataSource2" class="org.apache.commons.dbcp2.BasicDataSource"  
		    destroy-method="close">  
		    <property name="driverClassName" value="${driverClassName}" />  
		    <property name="url" value="${url}" />  
		    <property name="username" value="${username}" />  
		    <property name="password" value="${password}" />  
		    <!-- 连接初始值，连接池启动时创建的连接数量的初始值  默认值是0 -->  
		    <property name="initialSize" value="3" />  
		    <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请  默认值是0 -->  
		    <property name="minIdle" value="3" />  
		    <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 ，0时无限制  默认值是8 -->  
		    <property name="maxIdle" value="5" />  
		    <!-- 连接池的最大值，同一时间可以从池分配的最多连接数量，0时无限制   默认值是8 -->  
		    <property name="maxTotal" value="15" />  
		</bean> 
3. 注意和dbcp的区别：maxActive改为了maxTotal
4. jdbc.properties代码：
		 # dbcp  
		 driverClassName=com.mysql.jdbc.Driver  
		 url=jdbc:mysql://localhost:3306/springtest01?useUnicode=true&characterEncoding=utf-8  
		 username=root  
		 password=password

**2)JavaConfig配置方式：**
1. javaConfig的两种写法，其实和上面的DBCPUtil的那两种写法差不多。
2. 简单的写法(没用序列化的)：
		@Configuration
		public class SqlConfig {
			@Bean
			private DataSource dataSource(){
				BasicDataSource dataSource=new BasicDataSource();
				dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/springtest?useUnicode=true&characterEncoding=utf-8");
				dataSource.setDriverClassName("com.mysql.jdbc.Driver");
				dataSource.setUsername("root");
				dataSource.setPassword("root");
				dataSource.setInitialSize(20);
				return dataSource;
			}
		}
3. 得到了DataSource对象就可以通过该对象Bean得到连接。
该对象一直存在，线程池就一直存在。

---
