---
title: MySQL学习笔记（六）：视图，备份还原与事务
date: 2018-03-24 20:59:47
tags: MySQL
categories: 数据库

---
## 0.学习准备
1. 学习资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》
2. 构建新表：
		CREATE TABLE m_stu1 LIKE te_stu3;
		-- 蠕虫复制
		INSERT INTO m_stu1 SELECT * FROM te_stu3;
		CREATE TABLE m_class1 LIKE te_class1;
		INSERT INTO m_class1 SELECT * FROM te_class1;
		ALTER TABLE m_class1 CHANGE classid id INT auto_increment NOT NULL;
---
## 1.视图基本操作
**1)视图的创建：**
1. 视图简介：
view:是一种有结构(仅有表结构)没结果(结构中不存在数据)的虚拟表。
虚拟表的结果来源不是自己定义，而是从对应的基表中产生（视图的数据来源）
2. 创建视图基本语法：
		Create View 视图名字 as select语句；
as不能省略，相当于给后面的select语句结果起了个别名。
select语句可以是：普通，连接，联合或子查询。
3. 创建**单表视图：基表来源只有一个表**
		CREATE VIEW v1 AS
		SELECT * FROM m_stu1;
		CREATE VIEW v2 AS
		SELECT * FROM m_class1;
4. 创建**多表视图：基表来源不止一个表**
		CREATE VIEW v3 AS
		SELECT * FROM m_stu1 AS s LEFT JOIN m_class1 AS c ON s.classid=c.id;
但是创建失败，提示错误`[Err] 1060 - Duplicate column name 'id'`,说明id重复，修改创建代码如下：
		CREATE VIEW v3 AS
		SELECT s.*，c.cla_name FROM m_stu1 AS s LEFT JOIN m_class1 AS c ON s.classid=c.id;
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-01.jpg)
基表有多张时字段不能重复。

**2)视图的查看：**
1. 查看视图的结果。
2. 视图是张虚拟表，表的所有查看方式都适用于视图：
`show tables [like]`/`desc 视图名`/`show create table 视图名`
3. 视图与表几乎相同，和表有一个关键字的区别:view
查看视图的创建语句时也可以使用：
		show create view 视图名；
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-02.jpg)
4. 视图一旦创建，系统会在视图对应的数据库文件夹下创建一个对应的结果文件：**frm文件**。
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-03.jpg)
5. 视图不会影响基表的存在，也不会影响基表的数据。

**3)使用视图：**
1. 使用视图**主要是为了查询**，将视图当做表查询即可。
视图可以查询数据，但是视图本身并没有数据。
2. 简单的查询：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-04.jpg)
3. 视图的执行本质就是执行了封装的select语句。
相当于Java中的封装方法，代码的复用。

**4)修改视图：**
1. 视图本身不可修改，但是视图本身的来源是可以修改的。
修改视图就是修改视图本身的来源语句(select语句)。
2. 修改语法：
		Alter view 视图名字 as 新的select语句；
如：
		ALTER VIEW v1 AS
		SELECT id,name,age,sex FROM m_stu1;
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-05.jpg)

**5)删除视图：**
1. 语法：
		drop view 视图名字;
2. 视图的删除只能用view，不能用`drop table`。
视图理论上可以随意删(最好别删)。但是表不能随意删。

**6)视图的意义：**
1. 视图可以节省SQL语句，将一条复杂的SQL语句使用视图进行保存，以后可以直接对视图进行操作。
2. 数据安全，视图操作是主要针对查询的，对视图结果进行进行处理（删除），不会影响基表数据（相对安全）。
3. 视图往往在大项目中使用，而是在多系统中使用：可以对外提供有用的数据，但是隐藏关键（无用）的数据：数据安全。
4. 视图可以对外提供友好性：不同的视图提供不同的数据，对外好像专门设计的。
5. 视图可以更好（更容易）进行权限控制。

---
## 2.视图的数据操作
视图是可以对数据进行操作的，但是有很多限制。
将数据直接在视图上进行操作。
**1)新增数据：**
1. 数据新增就是直接对视图进行进行数据新增：
	- 多表视图不能数据新增。
	- 可以向单表中插入数据，但是视图中包含的字段必须包含基表中所有不能为空（或有默认值）的字段。
2. 测试：
往多表插入：
		INSERT INTO v3 VALUES(11,'0000000011','学生11',20,175,'男',3,'班级三');
往单表视图插入数据（不包含不能为空的code字段）：
		INSERT INTO v1 VALUES(11,'学生11',20,'男');
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-06-1.jpg)
说明视图是可以向基表插入数据的。
3. 修改v1的视图结构并插入：
		ALTER VIEW v1 AS
		SELECT * FROM m_stu1;
		INSERT INTO v1 VALUES(11,'0000000011','学生11',20,175,'男',3);
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-07.jpg)

**2)删除数据：**
1. 多表视图不能删除数据（连接）。
单表视图可以删除数据。
2. 测试：
多表视图删除数据：
		DELETE FROM v3 WHERE id=11;
单表视图删除数据：
		DELETE FROM v1 WHERE id=11;
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-08.jpg)
3. 视图可以操作数据，但是一般情况下不允许操作。

**3)更新数据：**
1. 理论上不论单表视图或者多表视图都可以更新数据库。
但是我的Mysql版本不能修改多表视图（可能某个版本之后不能修改了）
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-09.jpg)
单表更新：
		UPDATE v1 SET Height=178 WHERE id=5;
会发现原表的数据也一起修改了。
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-10.jpg)
2. **更新限制：**
`with check option`,如果在新增的时候限定了某个字段有限制，那么在对视图进行数据更新操作时数据会进行验证：
**保证更新之后该字段的数据依然可以被视图查询出来，否则不让更新。**
3. 限制测试：
创建限制视图：
		CREATE VIEW v4 AS
		SELECT * FROM m_stu1 
		WHERE age>25 WITH CHECK OPTION；
`WITH CHECK OPTION`限制视图更新时不能将已经得到的数据（age>25）的改为年龄小于25。
		UPDATE V4 SET age=24 WHERE id=6;
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-11.jpg)
可以修改数据为视图可以查到的。
		UPDATE V4 SET age=26 WHERE id=6;
也可以修改视图中没有的数据让视图可以查看的：
		-- 可以操作，但是无效果
		UPDATE V4 SET age=26 WHERE id=5;
只能操作看到的数据。

**4）视图算法：**
1. 需求：
查询每个班身高最高的学生。（使用视图解决）
原来的方法：
		SELECT * FROM (SELECT * FROM m_stu1 ORDER BY Height DESC) as stu GROUP BY classid;
错误的修改方法：
		CREATE VIEW v5 AS
		SELECT * FROM m_stu1 ORDER BY Height DESC;
		SELECT * FROM v5 GROUP BY classid;
这条方法的执行结果与
		SELECT * FROM m_stu1 GROUP BY classid ORDER BY Height DESC;
效果是一样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-12.jpg)
这时候就需要视图算法了。
2. **视图算法**
系统对视图以及外部查询视图的select语句的一种解析方式。
2. 视图算法分类：
分为三种：
	- Unserfined：未定义，默认的，不是一种实际使用的算法，告诉系统没有定义算法，让系统自己选择下列的算法：
	- `Temptable`：临时表算法，表示系统应该先执行视图的select语句，后执行外部的查询语句。
	- `Merge`：上面使用的算法，合并算法，系统先将视图对应的selec语句与外部查询视图的select语句合并，然后再执行。（效率高，但有时不准确），系统经常选择merge。
3. 算法指定，在创建视图时使用`algorithm`指定:
		CREATE ALGORITHM=TEMPTABLE VIEW v6 AS
		SELECT * FROM m_stu1 ORDER BY Height DESC;
		SELECT * FROM v6 GROUP BY classid;
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-13.jpg)
4. 视图的算法选择：
如果select语句包含查询子句（五子句：where等）,且顺序比外部的子句靠后，则一定得选择Temptable。
其他情况下选择默认或Merge。

---
## 3.数据备份
### 备份简介及存储引擎
**1)备份还原简介：**
1. 备份：将当前已有的数据或者记录保留
还原：将已保留的数据恢复到对应的表中
2. 为什么要备份还原（必要性）：
	- 防止数据丢失，被盗，误操作等；
	- 保护数据记录。
3. 数据备份还原的操作：
数据表备份，单表数据备份，SQL备份，增量备份。

**2)存储引擎概述：**
1. 详情见[MySQL学习笔记（三）](https://zjxkenshine.github.io/2018/03/16/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E4%B8%89%EF%BC%89/)
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-14.jpg)
一般只有Myisam和InnoDB是免费的。
2. 对比Myisam和InnoDB的数据存储方式:
	- InnoDB：只有表结构,数据全部存储到ibdata1文件中。（.ibd）
	- Myisam：表，数据，索引全都单独存储

### 文件备份与单表备份
**1)数据表(文件)备份：**
1. 不需要通过SQL来备份：
直接进入到数据库文件夹，复制对应的表结构以及数据文件。以后还原的时候直接将备份的内容放进去即可。
2. 数据表(文件)备份只要复制对应的结构与数据文件即可。
直接复制文件到其他数据库下即可。
(老版本的InnoDB不能这样备份，因为.ibd文件不是分离的)

**2)单表数据备份：**
1. 每次只能备份一张表，只能备份数据（表结构不能备份）。
2. 通常的使用：将表中的数据进行到导出到文件(一般是使用)。
3. 备份：从表中选出一部分数据保存到外部的文件中(outfile)：
语法：
		select */字段列表 into outfile 文件路径 from 数据表；
前提：外部文件不存在。
测设备份（备份学生表到D:\mysql\dump目录）：
		SELECT * INTO OUTFILE 'D:/mysql/dump/m_stu1.txt' FROM m_stu1;
打开目录下的`m_stu1.txt`，查看结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-15.jpg)
不要用记事本打开,否则编码改变数据会丢失。
注意要备份哪个数据库的表就要进入到哪个数据库环境下操作(`user database`)。

**3)单表数据高级备份：**
1. 可以自己指定一些字段和行的处理方式：
		select */字段列表 into outfile 文件路径 fields 字段处理 lines 行处理 from 数据表；
2. 参数说明：
	- fields 字段处理：
	`Enclosed by`:字段使用什么内容包裹,默认是控字符串
	`Terminated by`:字段以什么结束,默认'\r',tab键
	`Escaped by`:特殊符号用什么方式处理，默认是使用反斜杠转义
	- lines 行处理：
	`starting by`:每行以什么开始，默认是空字符串
	`Terminated by`:每行以什么结束，默认'/r /n'换行符
3. 测试，指定备份方式：
		-- 高级单表备份:
		SELECT * INTO OUTFILE 'D:/mysql/dump/m_stu2.txt'
		-- 字段处理方式
		FIELDS ENCLOSED BY '"'  -- 使用双引号包裹
		TERMINATED BY '|' -- 使用竖线分隔数据
		-- 行处理
		LINES STARTING BY 'start:'
		FROM m_stu1;
查看备份结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-16.jpg)

**4)单表数据还原：**
1. 还原：
讲一个在外部保存的数据重新恢复到表中
单表还原前提：
**表结构一定要存在。**
2. 语法：
		Load data infile 文件所在的路径 into table 表名（字段列表） fileds 字段处理 lines 行处理；
3. 测试：
		-- 模拟误删
		DELETE FROM m_stu1;
		-- 单表备份还原
		LOAD DATA INFILE 'D:/mysql/dump/m_stu2.txt'
		INTO TABLE m_stu1
		-- 字段处理方式
		FIELDS 
		ENCLOSED BY '"'  -- 使用双引号包裹
		TERMINATED BY '|' -- 使用竖线分隔数据
		-- 行处理
		LINES STARTING BY 'start:';
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-17.jpg)

---
## 4.数据备份（SQL备份与增量备份）
### SQL备份
**1)简介：**
最常用的备份方式，备份的是SQL语句，系统会对表结构以及数据进行处理，变成对应的SQL语句再进行备份。
还原只需要执行SQL语句即可(主要是针对表结构)。
2. SQL备份：**MySQL没有提供备份指令**，需要利用mysql提供的软件：**mysqldump.exe**（在mysql的bin目录下可以找到）

**2)mysqldump.exe使用：**
1. mysqldump.exe也是一种客户端，想要操作服务器，必须要认证。
		mysqldump[.exe] -hPup 数据库名字 [数据表名字1..][数据表名字2..] > 外部文件目录（.sql结尾）
2. 备份我的m_stu1到`D:/mysql/dump/m_stu3.sql`:
		mysqldump -uroot -p test02 m_stu1 > D:/mysql/dump/m_stu3.sql
备份成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-18.jpg)
但是会提示你这样是不安全的，但是只能这样操作（以我目前的认知）。
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-19.jpg)
还原的时候会先把原表删除后再创建添加。
3. 备份test02数据库：
		mysqldump -uroot -p test02 > D:/mysql/dump/test02.sql
时间会比较长,就不测试了。注意需要使用exit或quit退出mysql环境先。

**3)还原两种方式：**
1. 两种方式(单表与多表无区别)：
	- 1.使用`mysql.exe`客户端还原：
			mysql[.exe] -hPup 数据库名字 < 备份文件目录
	指定的数据库可以是别的数据库。
	- 2.使用SQL指令还原`Source`：
			Source 备份文件所在目录
2. 测试客户端方式还原：
		mysql -uroot -p test02 < D:/mysql/dump/m_stu3.sql
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-20-2.jpg)
也需要退出mysql环境才可以操作。
3. 测试SQL指令还原方式（需在数据库环境下）：
		DROP TABLE m_stu1;
		source D:/mysql/dump/m_stu3.sql;
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-21.jpg)

**4)SQL备份优缺点：**
优点：
1. 可以备份表结构和数据，操作方便

缺点：
1. 会浪费空间，备份会额外增加很多sql指令
2. 备份时间较长（一定要整表备份）

### 增量备份
1. 简介：
比较麻烦，视频中未详细介绍。
不是针对数据或者SQL指令进行备份，而是针对mysql服务器的日志文件进行备份。
2. 特点：
可以指定时间段进行备份，备份数据不会重复，而且**所有的操作都会备份。**
大项目一般都用增量备份。(还原数据精确)
3. 更多备份操作以后看书再写。

---
## 5.事务(手动控制)
**1)事务介绍：**
1. 适用情景：
*有一张银行账户表，A向B转账，A账户钱先减少，B用户钱后增加，但是A操作完之后断电了。A的钱减少了，但是B的钱没用增加。*
解决方案：
*A减少钱，但是不立即修改数据表，B收到钱之后，同时修改数据表。*
而负责这样做的一种机制----**事务**安全
2. 简介：
事务(transaction)：一系列要发生的连续的操作。
事务安全：一种保护连续操作同时满足(实现)的一种机制。
3. 事务安全的意义：
**保证数据操作的完整性。**
4. 数据库准备：
创建表并插入数据：
		-- 创建表
		CREATE TABLE m_account(
		id INT PRIMARY KEY auto_increment,
		number CHAR(16) NOT NULL UNIQUE COMMENT '账户',
		name VARCHAR(20) NOT NULL,
		money DECIMAL(10,2) DEFAULT 0.0 COMMENT '账户余额'
		)charset utf8;
		-- 插入数据
		INSERT INTO m_account VALUES(null,'0000000000000001','张三',1000),(null,'0000000000000002','李四',2000);
模拟张三给李四转1000元(数据减少):
		-- 模拟扣钱
		UPDATE m_account SET money=money-1000 WHERE id =1;
此时张三钱减少了，李四未收到钱，若这时退出了就会出现和刚开始的情况。
5. 事务操作的分类(两种)：
自动事务（默认的），手动事务

**2)手动事务：**
1. 开启事务：告诉系统以下所有的操作（写）不要直接写入到数据表，先存放到事务日志，data目录下的ib_log.file就是日志文件。
开启事务的命令：
		Start transaction
测试代码：
		-- 开启事务
		START TRANSACTION;
		-- 模拟李四借钱给张三，李四钱减少
		UPDATE m_account SET money=money-1000 WHERE id =2;
		-- 张三得到钱
		UPDATE m_account SET money=money+1000 WHERE id =1;
在一个窗口运行，并打开另一个窗口查看数据：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-22.jpg)
表并没有修改。（使用同一个窗口会看见修改但是实际并未修改）
2. 关闭事务：选择性的将日子文件中的结果保存到数据表（同步）或者说直接清空事务。
	- 提交事务：同步数据表（操作成功）
			COMMIT;
	- 回滚事务：直接清空日志表
			rollback;
3. 测试：
提交事务后再通过其他窗口查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-23.jpg)
已经修改了。此时再rollback已经没有意义了。
4. 事务要么提交，要么全部擦除。
5. **免费的存储引擎只有InnoDB支持事务。**

---
## 6.事务(原理及回滚点)
**1)事务原理：**
1. 事务开启后，所有的操作都会临时保存到事务日志，事务日志只有在得到commit指令才会同步到数据表，其他任何情况都会被清空（rollback,断开连接，断电）
2. 原理图：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-24.jpg)

**2)回滚点：**
1. 在某一个成功的操作完成之后，后续的操作有可能成功也可能失败，不管成功还是失败，前面的操作都已经成功，可以在当前成功的位置设置一个点，可以供后续失败操作返回到该位置，而不是返回所有操作。这个点称为回滚点。
在事务步骤很多的情况下经常用。
2. 语法
		-- 开启事务
		START TRANSACTION;
		-- 张三发工资
		UPDATE m_account SET money=money+10000 WHERE id =1;
		-- 设置回滚点
		SAVEPOINT sp1;
		-- 银行扣税
		UPDATE m_account SET money=money-10000*0.05 WHERE id =1;
		-- 回到回滚点
		ROLLBACK TO sp1;
		-- 继续操作
		UPDATE m_account SET money=money-10000*0.05 WHERE id =1;
		-- 查询结果 
		SELECT * FROM m_account;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-25.jpg)

---
## 7.事务(自动事务及事务特性)
**1)自动事务:**
1. 在mysql中默认的都是自动事务处理，用户操作完会立即同步到数据表中。
2. 自动事务的控制：`autocommit`变量
		show vairiables like 'autocommit'
关闭事务：
		set autocommit=off/0;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00020mysql%E5%AD%A6%E4%B9%A06-26.jpg)
自动关闭后需要手动来选择处理：
		commit/rollback
3. 通常都会使用自动事务。
开启事务：
		set autocommit=on/1;

**2)事务的特性：**
1. 事务有四大特性：ACID
A:Atomatic，原子性，事务的整个操作是一个整体，不可分割，要么全部成功，要么全部失败。
C:Consistency，一致性，事务操作前后，表中的数据没有变化。
I:Isolation，隔离性，事务操作是相互隔离不受影响的。
D:Durability,持久性，数据一旦提交，不可改变，永久改变数据表数据。
2. 锁机制：
InnoDB，默认是行锁，但是如果在事务过程中，没有用到索引，那么系统会自动全表检索，自动省纪委表锁。
行锁：只有当前行被锁住，别的用户不能操作。
标锁：整张表被锁住，别的用户都不能碰。
3. 何时使用事务：和钱有关的操作会用手动事务。
其余时候都是自动事务。

**3)事务的操作时数据的操作：**
删除数据表等针对结构的操作不可逆。

---
