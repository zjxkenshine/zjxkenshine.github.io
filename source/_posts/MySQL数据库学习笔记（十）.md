---
title: MySQL学习笔记（十）：SQL分区及一些常用的使用技巧
date: 2018-5-8 11:12:25  
tags: 
- MySQL
- 正则表达式
categories: 数据库

---
## 0.学习准备
1. 参考资料：
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
传智播客的视频学完了

2. 简单目录：
	- SQL分区
	- SQL分区类型
	- SQL分区管理
	- 正则表达式的使用
	- 其他一些函数的使用技巧

---
## 1.SQL分区概述及优点
**1)简介与优点：**
1. 分区：
指根据一定的规则，数据库吧一个表分解成多个更小的、更容易管理的部分。
就访问数据库而言，逻辑上而言只有一个表或者一个索引，但是实际上这个表可能由数个物理分区对象组成。
2. 每个分区对象都是一个独立的对象，可以作为表的一部分进行处理，分区对应用来说完全透明，不影响应用的业务逻辑。
3. SQL分区的主要优点包括以下四个方面：
	- 和单个磁盘或者文件系统分区相比，可以存储更多的数据。
	- 优化查询：
		- where子句只要扫描一个或者几个分区就可以。
		- SUM(),CONUNT()等函数查询时可以在每个分区处理再汇总。
	- 对于已经过期或者不需要保存的数据可以通过删除分区进行快速删除。
	- 跨多个磁盘来分散数据查询可以获得更大的查询吞吐量。
4. 分区与分表分库不是一个概念。
5. 就现在而言，“业内进行一些技术交流的时候也更多的是自己分库分表，而不是使用分区表。”
所以分区表仅仅学习。

**2)分区概述：**
1. 分区有利于管理非常大的表。引入了分区键。
2. 分区键用于根据某个规则，让数据根据规则分布在不同的分区中。
(区间值(或者范围值)、特定值列表或者HASH函数值执行时间的聚集)
3. 查看当前的MySQL是否支持分区：5.6之前
		SHOW VARIABLES LIKE '%partition%';
或者使用：
		SHOW VARIABLES LIKE 'have%';
查看是否有`have_partitioning`，我的Mysql8.0不支持分区功能：
![](http://p5ki4lhmo.bkt.clouddn.com/00057mysql%E5%AD%A6%E4%B9%A010-01.jpg)
4. 使用`SHOW PLUGINS`命令可以查看是否安装了分区插件。（Mysql5.6之后）
如果有partition插件则支持分区。
或者使用代码：
		SELECT
			PLUGIN_NAME as Name,
			PLUGIN_VERSION as Version,
			PLUGIN_STATUS as Status
		FROM
		INFORMATION_SCHEMA.PLUGINS
		WHERE
		PLUGIN_TYPE='STORAGE ENGINE';
在Mysql5.7下查看，发现支持分区：
![](http://p5ki4lhmo.bkt.clouddn.com/00057mysql%E5%AD%A6%E4%B9%A010-02.jpg)
5. 因为一般不会使用分区表，所以只是简单学习一下吧。

**3)创建一个分区表及注意事项：**
1. 创建一个拥有4个分区的表：
		-- 创建表并分为4个区
		CREATE TABLE m_part(part_id INT,name VARCHAR(20))
		ENGINE=INNODB
		PARTITION BY HASH(part_id)
		PARTITIONS 4;
使用的是HASH分区，依据的字段是part_id。
![](http://p5ki4lhmo.bkt.clouddn.com/00057mysql%E5%AD%A6%E4%B9%A010-03.jpg)
如果没有包括一个PARTITIONS子句，那么分区的数量将默认为1。
如果是集群则和集群的节点数相同。
2. 要使用HASH分区来分割一个表，要在CREATE TABLE 语句上添加一个“PARTITION BY HASH (expr)”子句，其中“expr”是一个返回一个整数的表达式。它可以仅仅是字段类型为MySQL整型的一列的名字。
3. Mysql分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区，反过来也一样。
4. 同一个表的不同分区的存储引擎要相同。

---
## 2.分区的类型
简单目录：
- 分区类型简介
	- 四种基本分区类型
	- 注意事项
- 分区类型测试：
	- RANGE分区
	- List分区
	- Columns分区
	- Hash分区
	- 线性Hash分区
	- Key分区

**1)分区类型**
- RANGE分区：
	基于属于一个给定连续区间的列值，把多行分配给分区。
- LIST分区：
	类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。
- HASH分区：
	基于**用户定义的表达式的返回值**来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式。
	（PARTITION BY HASH(能产生整数的表达式)）
- KEY分区：
	类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。
<br>
1. 分区键类型：
HASH分区键必须是INT类型或者能通过表达式产生int类型。
RANGE与LIST分区在5.5版本以上支持非整数类型分区键。
2. 关于主键，唯一键：
无论那种分区类型，要么分区表上没有主键或者唯一键，否则分区表的主键/唯一键都必须包含分区键。
（不能使用主键/唯一键之外的其他字段分区）
3. 关于分区名：
遵循Mysql命名规范，不区分大小写（无论那种操作系统）
表或者数据库命名的大小写敏感由操作系统决定（Windows不区分，Unix/Linux区分大小写）

**2)Range分区：**
1. 简单创建Range分区语句：
		CREATE TABLE employees (
			id INT NOT NULL,
			name VARCHAR(30),
			age INT NOT NULL
		)
		partition BY RANGE (age) (
			partition p0 VALUES LESS THAN (20),
			partition p1 VALUES LESS THAN (30),
			partition p2 VALUES LESS THAN (40),
			partition p3 VALUES LESS THAN (50)
		);
这样就根据年龄将该表分为了四个区。
2. Range分区语法：
利用取值范围将数据分成分区，区间要求连续，使用`partition p(n) VALUES LESS THAN (m)`进行分区定义。
设定分区的最大值。
3. 注意超出范围的错误：
如有一个年龄为55岁的员工，则插入数据时会报错。超出了分区范围。
![](http://p5ki4lhmo.bkt.clouddn.com/00057mysql%E5%AD%A6%E4%B9%A010-04.jpg)
添加分区：(也可以修改原有分区而不添加)
		ALTER TABLE employees ADD PARTITION (PARTITION P4 VALUES LESS THAN MAXVALUE);
再次添加：
![](http://p5ki4lhmo.bkt.clouddn.com/00057mysql%E5%AD%A6%E4%B9%A010-05.jpg)
4. Mysql5.1只支持整数列分区，而Mysql5.5之后支持非整数列分区。
可以使用日期或者字符串作为分区键进行分区。
5. RANGE分区功能适用情况：
	- 过期数据只要使用`alter table tb drop partition pn`来删除某一过期分区的数据就可以了。
	- 经常运行包含分区键的查询，MySQL可以很快确定哪个或那几个分区需要扫描。

**3)List分区：**
1. 简单创建测试：
		CREATE TABLE employees2(
			id INT NOT NULL,
			name VARCHAR(30),
			sex INT NOT NULL
		)
		partition BY LIST(sex) (
			partition p0 VALUES IN (1,2),
			partition p1 VALUES IN (3,4)
		);
2. List分区有很多和Range分区相似的地方，注意分区使用的是一个枚举的列表，而不是一个连续的区域。
`partition p0 VALUES IN (列表),`
3. Mysql5.5之后支持非整数分区，非整数的字段不需要额外的处理。

**4)Columns分区：**
1. Mysql5.5引入的分区类型，为了解决5.5版本之前RANGE和LIST只支持整数分区。
可以细分为两类：
	- RANGE Columns分区
	- LIST Columns分区
2. 这两个分区都支持以下几种数据类型：
	- 所有的整数类型。（小数类型不支持）
	- 日期时间类型：date和datetime
	- 字符串类型：char,varchar,binary和varbinary
3. Columns分区还支持多列分区：
RANGE COLUMNS：
		CREATE TABLE m_num(
			a INT,
			b INT
		)
		PARTITION BY RANGE COLUMNS(a,b)(
			PARTITION p0 VALUES LESS THAN (0,10),
			PARTITION p1 VALUES LESS THAN (10,20),
			PARTITION p2 VALUES LESS THAN (10,MAXVALUE),
			PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE)
		);
先根据A列排，再根据B列排。
LIST Columns也支持多列：
		CREATE TABLE m_num(
			a INT,
			b INT
		)
		PARTITION BY LIST COLUMNS(a,b)(
			PARTITION p0 VALUES IN ((0,10),(1,2)),
			PARTITION p1 VALUES IN ((10,20),(20,30),(30,40))
		);


**5)HASH分区：**
1. 分为两种：
	- 常规HASH分区：取模运算。HASH
	- 线性HASH分区：使用的是一个线性的2的幂运算法则。Linear HASH。
2. 常规hash分区：
		CREATE TABLE m_num(
			a INT,
			b VARCHAR(10)
		)
		PARTITION BY HASH(a)
		PARTITIONS 4;
3. 线性hash分区：
		CREATE TABLE m_num(
			a INT,
			b VARCHAR(10)
		)
		PARTITION BY LINEAR HASH(a)
		PARTITIONS 4;
4. 常规哈希分区与线性hash分区的优点：
	- 常规hash分区：数据平均分布，但是分区管理（增加或删除改动分区）时需要移动非常多的数据。
	- 线性hash分区：分区维护时可以更加迅速的处理，但是分布不均匀。
5. 当分区个数为2^n时，两种hash分区的结果相同。
6. HASH分区能够有效的分散热点。

**6)KEY分区：**
1. 非常类似于HASH分区，区别如下：
	- HASH分区允许使用自定义的表达式，而KEY分区只能使用Mysql提供的HASH函数。
	- HASH分区只支持整数分区，而KEY分区支持除了TEXT,BLOB外的任何数据类型。
2. 同样，也分为两种：(和hash的一样)
	- 常规key分区：KEY
	- 线性key分区：LINEAR KEY
3. 常规KEY分区：
		CREATE TABLE m_num(
			a INT NOT NULL,
			b VARCHAR(10)
		)
		PARTITION BY key(a)
		PARTITIONS 4;
如果使用`PARTITION BY key()`则会优先使用主键作为分区键，如果没有主键则会选择非空唯一键作为分区键，也没有非空唯一键则报错。
4. 线性KEY分区：
		CREATE TABLE m_num(
			a INT NOT NULL,
			b VARCHAR(10)
		)
		PARTITION BY LINEAR KEY(a)
		PARTITIONS 4;

---
## 3.分区管理
**1)子分区及分区NULL值处理**
1. 子分区又称作复合分区，指在每个分区表中对每个分区再次进行分割。
Mysql支持外层分割可以是RANGE或者LIST,子分割可以是HASH或者KEY。
2. 子分区例子：
		CREATE TABLE m_num(
			a INT NOT NULL,
			b VARCHAR(10)
		)
		PARTITION BY RANGE(a)
		SUBPARTITION BY HASH(a)
		SUBPARTITION 2
		(
			partition p0 VALUES LESS THAN (20),
			partition p1 VALUES LESS THAN (30),
			partition p2 VALUES LESS THAN (MAXVALUE),
		);
m_num先进行RANGE分区再进行HASH分区，一共分为了3\*2=6个分区。
使用`SUBPARTITION BY HASH/KEY()`可以对子分区进行分配。
3. 关于NULL值：
MYSQL允许分区键为NULL，不同的分区方式有不同的处理：
	- Range:作为最小值处理
	- LIST则会报错
	- HASH和KEY当做0处理
4. 最好使用非空或者默认值来绕过NULL值，以免产生不必要的错误。

**2)RANGE&LIST分区管理**
1. 添加，删除，重新定义分区的处理上，RANGE分区和LIST分区非常相似。
2. 删除分区：
		ALTER TABLE table_name DROP PARTITION part_name;
3. 增加分区：
		ALTER TABLE table_name ADD PARTITION
RANGE分区：(注意n一定要比当前最大值大)
		ALTER TABLE table_name ADD PARTITION(PARTITION pn VALUES LESS THAN (n));
LIST分区：(值a,b,c不能在现有分区值中存在)
		ALTER TABLE table_name ADD PARTITION(PARTITION pn VALUES IN (a,b,c));
4. 拆分分区：
RANGE分区：(将pn拆分为了三个分区)
		ALTER TABLE table_name REORGANIZE PARTITION pn INTO(
			PARTITION pn VALUES LESS THAN(n1),
			PARTITION pn+1 VALUES LESS THAN(n2),
			PARTITION pn+2 VALUES LESS THAN(n3)
		);
LIST分区：
		ALTER TABLE table_name REORGANIZE PARTITION pn INTO(
			PARTITION pn VALUES IN (a,b,c),
			PARTITION pn+1 VALUES IN (d,e,f),
			PARTITION pn+2 VALUES IN (g)
		);
5. 合并分区：
RANGE分区：
		ALTER TABLE table_name REORGANIZE PARTITION p1，p2,p3 INTO(
			PARTITION pn VALUES LESS THAN(n),
		);
LIST分区：
		ALTER TABLE table_name REORGANIZE PARTITION p1，p2,p3 INTO(
			PARTITION pn VALUES IN (a,b,c,d,e,f,g)
		);
6. 重新定义分区（拆分，合并分区）的注意事项：（RANGE分区和LIST分区相同）
	- 只能重新定义相邻的分区，重新定义后的区间需要和原来相同
	- 不能使用重新定义分区改变分区类型，如RANGES变为HASH

**3)HASH&KEY分区管理**
和RANGE与LIST不同，HASH和KEY的分区只有删除和新建两种操作，而且操作方式也和RANGE与LIST不同。
1. 删除分区的操作：不是真正意义上的删除，而是合并
		ALTER TABLE table_name COALESCE PARTITION num;
num是删除到多少个分区，如果大于当前分区数则会报错。
该命令不能用于新增分区。
2. 增加分区数：不是真正意义的增加，只是拆分
		ALTER TABLE table_name ADD PARTITION n;
增加n个分区，而不是增加到n个分区。

---
## 4.分区表的局限与限制
1. 禁止构建：
分区表达式不支持以下几种构建：
存储过程，存储函数，UDFS或者插件
声明变量或者用户变量
可以参考分区不支持的SQL函数
2. 算术和逻辑运算符
分区表达式支持+,-,*算术运算，但是不支持DIV和/运算（还存在，可以查看Bug #30188, Bug #33182）。但是，结果必须是整形或者NULL（线性分区键除外，想了解更多信息，可以查看分区类型）。
分区表达式不支持位运算：|，&，^，<<，>>，~ .
3. HANDLER语句
在MySQL 5.7.1之前的分区表不支持HANDLER语句，以后的版本取消了这一限制
4. 服务器SQL模式
如果要用用户自定义分区的表的话，需要注意的是，在创建分区表时的SQL模式是不保留的。可能会导致严重的错误。
一旦SQL模式在创建分区表后改变，可能导致这些表的行为发生重大变化，很容易导致数据丢失或者损坏。
强烈建议创建分区表后千万不要修改服务器的SQL模式。
5. 更加详细的关于分区表的限制参考博客：
[MySQL分区表的局限和限制详解](http://www.jb51.net/article/108165.htm)

---
## 5.正则表达式的使用




---