---
title: MySQL学习笔记（九）：SQL安全，SQLMode，表锁及事务补充
date: 2018-5-1 19:25:53 
tags: MySQL
categories: 数据库

---
## 0.学习准备
1. 参考资料：
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
传智播客的视频学完了

2. 简单目录：
	- SQL中的安全问题简介
	- SQL Mode及相关问题简介
	- 锁机制简介及简单使用LOCK
	- 事务的复习及补充
	- Linux下安装MySQL

---
## 1.SQL中的安全问题
SQL语句操作不当会给系统造成很大的安全隐患，最重要的就是SQL注入。

**1)SQL注入简介：**
1. SQL：结构化查询语言，是一种和数据库交互的文本语言。
2. SQL注入：
利用某些数据库的外部接口将用户数据插入到实际的数据库操作语言(SQL)中，从而到达入侵数据库乃至操作系统的目的。
3. SQL注入的危害：
攻击者利用SQL注入读取，修改或者删除数据库内的数据。
获取数据库用户名和密码等敏感信息。甚至可以获得数据库管理员的权限干任何事情。
无法通过简单的配置进行自我保护，一般的防火墙无法拦截。
4. 所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。
通过WEB表单递交查询字符特别容易受到SQL注入攻击。

**2)其他可能会出现的MySQL安全问题：**
1. 防止sql注入，对特殊字符进行转义(addslashes)，或者使用已经编译好的sql语句进行变量的绑定；
2. 当sql运行出现错误的时候，不要把数据库返回的错误信息全部显示给客户，以防止泄漏服务器和数据库的相关信息；
3. 最小权限原则，特别不要使用root用户，为不同的类型的动作或者组建不同的账户；

---
## 2.应对SQL注入的措施
主要有三种方式：
- PreparedStatement绑定变量
- 使用应用程序提供的转换函数
- 自己定义函数进行校验

**1)PreparedStatement绑定变量(推荐)：**
1. MySQL服务端并不存在共享池的概念，所以在MySQL上使用绑定变量就可以避免SQL注入，增加安全性。
2. 如下JDBC简单代码：
		Class.forname("com.mysql.jdbc.Driver").newInstance();
		String url="jdbc:mysql://127.0.0.1:3306/test";
		String user="username";
		String password="password";
		Connection conn = DriverManager.getConnection(url,user,password);
		String sqlstmt="select * from user where username=? and password=?";
		PreparedStatement preStmt=conn.prepareStatement(sqlStmt);
		preStmt.setString(1,"user1 'or 1=1'");	//这里的or 1=1就是注入攻击的内容
		preStmt.setString(2,"password");
		Resultset rs=preStmt.executeQuery();
		while(rs.next()){
			//对查询出的数据的操作
		}
3. 常见的SQL攻击方式还有：
`preStmt.setString(1,"user1 '/*'" );`
和
`preStmt.setString(1,"user1 '#'" );`
or的那种方法时利用了SQL逻辑，而这两个攻击则是想要直接注释掉后面的SQL代码。
4. 无论那种SQL注入，PrepareStatement都能够很好的防御，使用绑定变量，可以正确解析单引号，里面的攻击字符串不会被解析为条件。从而达到了防止SQL注入的目的。
5. 切记不要用拼接字符串。
6. 原理：
sql注入只对sql语句的准备(编译)过程有破坏作用
而PreparedStatement已经准备好了,执行阶段只是把输入串作为数据处理,
而不再对sql语句进行解析,准备,因此也就避免了sql注入问题.

**2)使用应用程序提供的转换函数：**
很多应用程序都提供了对特殊字符创进行转换的函数，但是似乎没有针对Java的：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-01.jpg)

**3)自己定义函数进行校验：**
1. 简单来说就是过滤非法字符。可以有以下几种方式：
	- 整理数据使之变得有效
	- 拒绝已知的非法输入
	- 只接受已知的合法输入
2. 最好的方式就是对可能出现攻击的数据进行简单分类(根据非法字符串)，然后分别用正则表达式对用户的输入数据进行严格的检测和验证，一定要严格。
3. 书上的方法示例(PHP版)：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-02.jpg)
4. 更多的信息可以查看博客：
<https://blog.csdn.net/zhangyongqiang123/article/details/52768730>

---
## 3.SQL Mode简介
**1)SQL Mode(SQL模式)简介：**
1. SQL Mode定义了MySQL应支持的SQL语法、数据校验等。
这样可以更容易地在不同环境中使用MySQL。
MySQL的一个特有的特性就是可以运行在不同的SQL Mode下。
2. 在MySQL中SQL Mode常用来解决下面几类问题：
	- 通过设置SQL模式可以完成不同严格程度的数据校验，有效保障数据的有效性。
	- 通过设置SQL模式为ANSI模式，来保证大多数SQL符合标准的SQL语法，这样应用在不同的数据库之间进行迁移时就不需要对业务的SQL进行较大的修改。
	- 在不同的数据库之间进行迁移时可以设置SQL模式使MySQL上的数据更方便的迁移到目标数据库中。

**2)SQLMode的常见的值：**
1. 常用的模式组合(由多个原子模式组成)
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-05.jpg)
	- ANSI：标准模式
	- STRICT_TRANS_TABLES：严格模式
	- TRADITIONAL：也是一种严格模式
2. 部分原子模式(sql_mode值)的介绍：
- ONLY_FULL_GROUP_BY：
对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么将认为这个SQL是不合法的。
- STRICT_TRANS_TABLES：
在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做任何限制
- NO_ZERO_IN_DATE：
在严格模式，不接受月或日部分为0的日期。如果使用IGNORE选项，我们为类似的日期插入'0000-00-00'。在非严格模式，可以接受该日期，但会生成警告。
- NO_ZERO_DATE：
在严格模式，不将 '0000-00-00'做为合法日期。你仍然可以用IGNORE选项插入零日期。在非严格模式，可以接受该日期，但会生成警告
- ERROR_FOR_DIVISION_BY_ZERO：
在严格模式，在INSERT或UPDATE过程中，如果被零除(或MOD(X，0))，则产生错误(否则为警告)。如果未给出该模式，被零除时MySQL返回NULL。如果用到INSERT IGNORE或UPDATE IGNORE中，MySQL生成被零除警告，但操作结果为NULL。
- NO_AUTO_CREATE_USER：
防止GRANT自动创建新用户，除非指定了密码。
- NO_ENGINE_SUBSTITUTION：
如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常

**3)SQLMode设置与测试：**
1. 创建测试表m_people：
		CREATE TABLE `m_people` (
		  `id` int(11) NOT NULL AUTO_INCREMENT,
		  `name` varchar(10) DEFAULT NULL,
		  PRIMARY KEY (`id`)
		) ENGINE=InnoDB DEFAULT CHARSET=utf8
使用desc查看表的定义：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-03.jpg)
2. 使用`select @@sql_mode`查看当前数据库使用的SQL Mode：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-04.jpg)
当前的模式为STRICT_TRANS_TABLES，NO_ENGINE_SUBSTITUTION。
3. 在该模式下插入一个超过长度的值：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-06.jpg)
超过长度会出错。
4. 修改模式为ANSI标准模式:
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-07.jpg)
这种修改方式只在本次连接有效。
5. 再次插入一个超过长度的数：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-08.jpg)
出现了警告，进行了截断后存储，但是不报错。
6. 修改SQL Mode的命令：
`SET [SESSION|GLOABAL] sql_mode='模式';`
SESSION表示只在当前连接有用，GLOABAL表示对全部新的连接有用(但是对当前连接没用)。
也可以在MySQL启动时通过`--sql-mode="模式"`来设置。
也可以通过配置文件设置:vim /etc/my.cnf
在my.cnf（my.ini）添加如下配置:
[mysqld]
sql_mode='你想要的模式'

---
##	4.SQL Mode功能与迁移时的使用
创建表m_modetest用于测试：

	CREATE TABLE `m_modetest` (
	  `context` varchar(255) DEFAULT NULL,
	  `time` datetime DEFAULT NULL
	) ENGINE=InnoDB DEFAULT CHARSET=utf8

**1)SQL Mode常见功能：**
1. 校验日期数据的合法性：常用的功能
插入一个正确但是不完整的值：补齐为0
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-09.jpg)
插入一个错误的值：(5月50号)
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-10.jpg)
全部都变为0了，并且有一个警告。
设为严格模式：`SET SESSION sql_mode='TRODITIONAL'`
再次进行插入：报错
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-11.jpg)
2. 除数为0的情况的处理：
在数据库中添加一个int类型的字段：
		alter table m_modetest add column shang int;
在标准模式ANSI下：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-14.jpg)
设置为严格模式：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-15.jpg)
3. 任何模式下直接查询时除数为0结果都是NULL。
在标准模式ANSI下：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-12.jpg)
设置为严格模式，再进行查询：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-13.jpg)
4. 启用`NO_BACKSLASH_ESCAPES`模式，使反斜杠`\`变为普通字符。
如果数据中含有反斜杠字符，启用`NO_BACKSLASH_ESCAPES`保证数据的正确性。
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-16.jpg)
修改模式：（前面的是ASNI组合模式的内容）
`SET SESSION sql_mode='REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ONLY_FULL_GROUP_BY,ANSI,NO_BACKSLASH_ESCAPES';`
然后再次添加：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-17.jpg)
反斜杠已经被解析成字符了。
5. 启用`PIPES_AS_CONCAT`模式，将`||`解析为连接字符串操作：
在ANSI下默认启用了PIPES_AS_CONCAT模式：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-18.jpg)
未启用时PIPES_AS_CONCAT模式的输出是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-19.jpg)
主要是因为在其他数据库(如Oracle)中||就是连接字符串的操作，所以为了统一MySQL也加入了对应的模式。(最好就是使用ANSI模式)

**2)SQL Mode在迁移中的使用：**


---

