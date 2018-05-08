---
title: MySQL学习笔记（九）：SQL安全，SQLMode，表锁简介及分布式事务
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
	- 简单使用LOCK
	- 事务的复习及补充
	- 分布式事务
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
1. MySQL提供了很多种数据库的组合模式来满足在不同数据库之间迁移数据的需求。
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-20.jpg)
修改组合模式的方法和前面的相同。
2. 如NO_TABLE_OPTIONS就是可以去掉MySQL建表语句中的ENGINE关键字，使之成为标准的建表语句。

---
## 5.表锁的简单使用
**1)简介：**
1. MySQL支持对MyISAM和MEMORY的表使用表级锁定，对BDB的表使用页级锁定，对InnoDB存储引擎的表使用行级锁定。
2. 因为事务的完整性会涉及到锁，先简单学习一下。
具体的关于锁的知识以后再来学习。

**2)简单使用：**
1. 简单语法：
		lock tables table_name1 {read|write}[table_name2...];
		unlock tables;
lock tables可以给一个或者多个表上锁。读读不互斥，读写互斥，写写互斥。
unlock tables可以释放当前线程获得任何锁定
(当前线程获取其他锁或者连接关闭时锁也会自动释放)
2. 简单测试：
读读不互斥(两个客户端)：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-21.jpg)
在另一个客户端执行写操作时，会等待该表的锁。只有在第一个客户端释放锁之后才会继续执行。
读写互斥：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-22.jpg)
3. 书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-23.jpg)

---
## 6.事务控制(复习与补充)
详细的普通事务用法可以查看前面的笔记：
<https://zjxkenshine.github.io/2018/03/24/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E5%85%AD%EF%BC%89/>
**1)事务简介：**
1. 基本语法：
设置是否自动提交(默认自动提交事务，最好使用默认)：
		SET AUTOCOMMIT={0|1};
关闭自动提交事务之后要手动进行提交或者混回滚。
2. 在自动提交的情况下针对某些特定的操作需要使用事务控制，使用语法：
		START TRANSACTION|BEGIN;
		COMMIT [and [no] chain] [and [no] release];
		ROLLBACK [and [no] chain] [and [no] release];
3. 上面语法中的元素介绍：
	- `START TRANSACTION`或者`BEGIN`用于开启一个新的事务。
	- `COMMIT`/`ROLLBACK`：提交或者回滚事务。
	- `CHAIN`：默认未开启，提交或回滚后立即开启一个新事务，而且和刚才的事务有相同的隔离级别。
	- `RELEASE`：默认未开启，开启后提交或者回滚事务后会断开客户端连接。
	- 默认是在事务提交后回到自动提交的状态。
4. 关于基本的事务测试可以查看前面的笔记。
5. 在同一个事务中最好不要使用不同存储引擎的表，否则使用COMMIT或者ROLLBACK进行提交或者回滚时要额外处理，因为提交和回滚只能处理事务类型的表。
6. 和Orace一样，所有的DDL(数据定义语言)都是不能回滚的，并且部分的DDL会导致隐性提交。

**2)开启事务会造成表锁释放：**
1. 简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-24.jpg)
2. 可以看到右边的客户端等待了12秒。
3. 回滚事务或者提交事务不会释放锁。

**3)关于回滚点：**
1. 在事务中通过SAVEPOINT指定回滚事务的一部分，但是不能提交事务的一部分。
2. 可以指定相同名字的回滚点，但是后面的会覆盖前面的。
3. 删除回滚点：
		RELEASE SAVEPOINT;

---
## 7.分布式事务
MySQL从5.0.3开始支持分布式事务，但是分布式事务只支持InnoDB存储引擎(暂时)。

**1)分布式事务原理：**
1. 一个分布式事务会涉及多个行动，这些行动本身也是事务性的。所有的行动要么一起成功完成，要么一起回滚。(分布执行，一起提交)
2. 在MySQL中，使用分布式事务的应用程序涉及到一个或者多个资源管理器和一个事务管理器。
	- 资源管理器(RM)：用于提供通向事务资源的路径。如数据库服务器。
	必须要可以提交或者回滚所管理的事务。
	- 事务管理器(TM)：用于协调作为一个分布式事务一部分的事务。
	TM与管理每个事务的RMs进行通信。
	在分布式事务中，各个单个事务均是分布式事务的分支事务。事务的各个分支由唯一命名进行标识。
3. mysql在执行分布式事务（外部XA）的时候，mysql服务器相当于xa事务资源管理器，与mysql链接的客户端相当于事务管理器。
(分布式嘛，服务器肯定要多台喽)
4. 分布式事务原理：分段式提交
分布式事务通常采用2PC协议，全称Two Phase Commitment Protocol。该协议主要为了解决在分布式数据库场景下，所有节点间数据一致性的问题。分布式事务通过2PC协议将提交分成两个阶段：
	- 准备阶段：即所有的参与者准备执行事务并锁住需要的资源。参与者ready时，向transaction manager报告已准备就绪。
	- 提交阶段：当transaction manager确认所有参与者都ready后，向所有参与者发送commit命令或者rollback命令。 
5. 简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-25.jpg)
当TM发现RM只有一个时（单分支），事务可以同时进行预备和提交。

**2)分布式事务管理语法：**
1. 常用语法：
`XA {START|BEGIN} xid [JOIN|RESUME]`：启动xid事务(xid必须是一个唯一值)
`XA END xid [SUSPEND [FOR MIGRATE]]`：结束xid事务
`XA PREPARE xid`：准备、预提交xid事务
`XA COMMIT xid [ONE PHASE]`：提交xid事务
`XA ROLLBACK xid`：回滚xid事务
`XA RECOVER`：查看处于PREPARE阶段的所有事务
2. xid是一个XA事务标识符。
Xid由客户端提供或者由服务端自动生成。
xid包含1~3个部分：
		xid:gtrid[,bqual[,formatID]]
gtrid：分布式事务标识符，相同的分布式事务应该使用相同的gtrid。
bqual：分支限定符，默认为空字符串。一个分布式事务中的不同分支bqual需要是唯一的。
formatID：gtrid和bqual的使用情况，默认为1。
3. 简单过程：
启动事务-->执行各种语句-->准备事务-->提交/回滚事务
（单分支时准备事务可以不执行）

**3)存在的隐患：Mysql5.7修复**
1. 如果分支在准备阶段，服务器宕机会导致备份文件中的数据不全。(已修复)
2. 准备阶段客户端崩溃未准备的分支会回滚，准备的分支会提交，造成数据的不完整。(已修复)
3. 关于性能问题:(这个问题一直存在)
XA的性能很低。一个数据库的事务和多个数据库间的XA事务性能对比可发现，性能差10倍左右。因此要尽量避免XA事务，例如可以将数据写入本地，用高性能的消息系统分发数据。或使用数据库复制等技术。只有在这些都无法实现，且性能不是瓶颈时才应该使用XA。

---
## 8.centos7上安装Mysql8.0.11(初学版)
**1)下载解压：**
我是从Windows下载后再使用xftp传给Linux的。
也可以使用：

	wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-linux-glibc2.12-i686.tar.gz
1. 下载地址
<https://dev.mysql.com/downloads/mysql/>
选择自己想要的版本，然后在下面选择：Compressed TAR Archive版本
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-26-1.jpg)
64位选择第二个，32位选择第一个
2. 点击下载：
可能需要填写一个调查表，填写好就可以了。
3. 下载完成后使用xftp将下载的压缩包(mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz)传到Linux。
5. 解压：
		# tar -zxvf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
6. 设置一个软连接：
		# ln -s mysql-8.0.11-linux-glibc2.12-x86_64 mysql
7. 进入该目录发现解压后的mysql就已经可以使用了,但是还未安装数据库：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-27-1.jpg)
8. 将主要使用的命令mysql软连接到/usr/bin/目录：
		# ln -sf /usr/local/mysql/bin/* /usr/bin/

**2)修改用户用户组及初始化：**
1. 添加用户与用户组：
		#添加用户组
		groupadd mysql
		#添加用户mysql 到用户组mysql
		useradd -g mysql mysql
我的everything版本这些都设置好了。。。
2. 创建data文件用于存放数据：
		mkdir /usr/local/mysql/data
3. 修改文件夹所有者：
		cd /usr/local/mysql
		chown -R mysql:mysql ./
或者：
		chown -R mysql.mysql /usr/local/mysql/
改变用户组：
		chgrp -R mysql ./
修改权限：
		chmod 755 /usr/local/mysql -R
4. 初始化数据库：
		bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/
		bin/mysql_ssl_rsa_setup
这是候有一个默认密码得记下来。详情先看后面。

**3)配置文件的创建及配置：**
1. 在mysql/support-files目录下新建文件my-default.cnf：
		vim my-default.cnf
简单配置里面的内容如下：
		[mysqld]
		basedir = /usr/local/mysql
		datadir = /usr/local/mysql/data
		log-error=/usr/local/mysql/data/err.log
		prot = 3306
		socket = /tmp/mysql.sock
		character-set-srever=utf8
详细的配置参数可以查看：
<https://www.cnblogs.com/langdashu/p/5889352.html>
2. 复制配置文件到/etc/my.cnf
		cp -a ./support-files/my-default.cnf /etc/my.cnf
如果原来该文件，则覆盖掉。

**4)建立Mysql服务：**
1. 添加到系统服务
		cp -a ./support-files/mysql.server /etc/init.d/mysqld
		chmod +x /etc/rc.d/init.d/mysqld
		chkconfig  --add mysqld
2. 检查服务是否生效：
		chkconfig  --list mysqld

**5)配置全局环境：**
1. 编辑/etc/profile文件：
`# vim /etc/profile`
在 profile 文件底部添加如下两行配置，保存后退出
		PATH=/data/mysql/bin:/data/mysql/lib:$PATH
		export PATH
2. 使配置立即生效：
`# source /etc/profile`

**6)启动服务与相关配置：**
1. 启动MySQL服务：(停止为stop)
		# service mysqld start
或者：
		# /etc/init.d/mysqld start
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-28-1.jpg)
2. 但是这样是无法登录的，因为不知道密码。
3. 默认的error日志文件位置：
查了一天不知道日志文件在哪，然后发现将data文件删除后重新执行初始化操作就会看到默认密码了：jGqhoLL9w6+f
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-29-1.jpg)
然后发现日志文件在这里：(ps -ef | grep mysql也能看见只不过少了前缀)
`/usr/local/mysql/data/localhost.localdomain.err`
....(苦瓜脸)...
4. 登录：
使用`mysql -u root -p`登录:
输入刚刚的默认密码进入。
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-30.jpg)
激动。。。
但是有提示需要重新设置密码才能正常使用。
5. 修改密码：
		`alter user 'root'@'localhost' identified by "123456";`
将这个123456改为自己的密码就行了。
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-31.jpg)
大功告成，可以开始正常使用了。

---
## 9.配置远程连接客户端
1. 通过如下命令查看端口号：
		show global variables like 'port'；
如果直接连接该端口，则会失败，需要额外的配置。
2. 默认安装的Mysql数据库是没有对外开放的，所以需要额外的配置。

**1)配置Mysql对外开放：**
1. 切换到mysql数据库：
		use mysql;
2. 赋予权限：
		update user set host='%' where user='root' limit 1;
可以使用：
		select host from user where user='root';
来验证是否更新成功。
3. 刷新权限：
		flush privileges;
4. 检查3306端口是否开放：(无查询结果则未开防)
		netstat -nupl|grep 3306
或者使用：
		firewall-cmd --query-port=3306/tcp
5. 开发3306端口：
		firewall-cmd --add-port=3306/tcp

**2)连接时出现问题：**
1. 问题：
		Client does not support authentication protocol requested by server; consider upgrading MySQL client
2. 原因：
Mysql8.0.11使用了新的密码验证机制，而很多客户端还来不及更新，所以导致出错。
3. 解决方法：
		-- 切换数据库
		USE mysql;
		-- 更新密码
		ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
		-- 刷新权限
		FLUSH PRIVILEGES;
root是用户名，localhost是ip地址127.0.0.1都是特指本机，mysql_native_password是旧的密码验证机制，123456是密码，最后别忘了分号；
如果更新失败可能是前面更新了ip,使用这个：`'root'@'%'`
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-32.jpg)
4. 连接成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00055mysql%E5%AD%A6%E4%B9%A09-33.jpg)

---