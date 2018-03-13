---
title: MySQL学习笔记（一）:数据库基础知识介绍与基本操作
date: 2018-03-10 16:20:31
tags: MySQL
categories: 数据库

---
## 0.数据库再学习
以前一直用的都是简单的增删改查，是时候来一波高大上的操作了。
从删库到跑路！
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

---
## 1.数据库基础
1. 什么是数据库?
	- database，存储数据的仓库。
	- 高效的存储和处理数据的介质。（磁盘，内存）
2. 数据库的分类:
基于存储介质不同：关系型数据库（SQL）和非关系型数据库（NoSQL：not only SQL）
3. 数据库产品：
	- 关系型数据库
		- 大型:Oracle,DB2
		- 中型:SQL-SERVER,Mysql
		- 小型:access等
	- 非关系型数据库：
	memcached,mongodb,Redis
4. 关系型数据库和非关系型数据库的区别:
关系型数据库：安全（保存磁盘基本不可能丢失），容易理解，比较浪费空间（二维表）
非关系型数据库：效率，但是不够安全（丢失断电）

---
## 2.关系型数据库及关键字说明
1. 什么是关系型数据库？
建立在关系模型上(数学模型)的数据库。
2. 关系模型：建立在关系上的模型。
关系模型包括三个方面：
	- 数据结构:数据存储的问题，二维表（有行和列）
	- 操作指令:所有SQL语句
	- 完整性约束:表内数据约束（字段与字段），表与表之间的约束（外键）
3. 关系型数据库的设计：
从需要存储的数据需求中分析，如果是一类数据（实体）应该设计成一张二维表。
**表**：表由**表头**（字段名：规定数据的名字）和**数据部分**（存储数据的单元）组成。
4. 实际分析:
简单分析一个教学系统：
	1.找出实体类：讲师表，学生表，班级表
	2.找出实体中应该存在的数据信息
			讲师：姓名，性别，年龄，工资...
			学生：姓名，性别，学号，学科...
			班级：班级名称，教师编号
	3.关系型数据库：维护实体与内部，实体与实体之间的联系。
	实体内部联系：
	---每个学生都有姓名，性别，学号，学科信息
	---某一行所有字段都是描述一个学生（内部联系），某一列只能放性别（内部约束）
	实体与实体之间的联系:
	---每个学生肯定属于某个班级，每个班级一定有多个学生（一对多）
	---在学生表内增加一个班级字段来指向班级
5. 关键字说明：
数据库：database(DB)
数据库系统：DBS（Database System）:是一种虚拟系统，将多种内容关联起来称呼
DBS=DBMS+DB
DBMS：Database Management System:数据库管理系统，专门管理数据库
DBA：Database Administator,数据库管理员
= - =
行/记录（row/record）：本质是一个东西，指的都是表中的一行，行是从结构角度出发，记录是从数据角度出发。
列/字段（column/field）：本质都是数据库中的一列数据

---
## 3.SQL
1. SQL：Structed Query Language,结构化查询语言（数据以查询为主:99%是在进行查询操作）
2. SQL分为三个部分:
	- DDL:Date Definiton Language,数据定义语言用来维护存储数据的结构（数据库，表）
	代表指令:**creat,drop,alter**等
	- DML：Data Maniplation Language,数据操作语言，用来对数据进行操作（数据表中的内容），代表指令：i**nsert，delete，update**等。其中DML内部又单独进行了一个分类：
	---DQL:Data Query Language,数据查询语言，如**select**
	---DCL:Data Control Language,数据库控制语言，主要负责管理权限（用户），代表指令：**grank,revoke**等。
3. SQL是关系型数据库的操作指令，SQL是一种约束（类似W3C）,不同的数据库产品（如Orcal,mysql）内部会有一些细微差别。

---
## 4.MySQL
1. MySQL是一种c/s结构的软件:客户端/服务器，若想访问服务器必须通过客户端（服务器一直运行，客户端在需要的时候运行）
2. MySQL的安装与配置：
参考地址：
<https://jingyan.baidu.com/article/cd4c2979033a17756f6e6047.html>
3. 交互方式：
--1.客户端连接认证，连接服务器。
开启服务：`mysqld --console`或`net start mysql`
root用户登录：`mysql -u root -p`
--2.发送SQL
--3.服务器接收SQL指令，处理SQL指令，返回操作结果
--4.客户端接收结果，显示结果
`show databases;`(注意分号)
![](/img/00010mysql学习笔记一1.jpg)
--5.断开连接（释放资源，服务器并发限制）
`exit`,`quit`或`\q`。
4. Mysql服务器对象：
没有办法完全了解内部的内容：只能粗略分析数据库服务器的内部结构。
将Mysql服务器内部对象分成了四层：
系统（DBMS）>数据库（DB）>数据表（Table）>字段（field）
windows>mysqld.exe>DB>Table>filed

----
## 5.SQL基本操作
1. 基本操作：CRUD
按操作对象进行分类，分为三类：库操作，表操作，数据操作
2. 在cmd中注释
`-- 注释`,也可以用`#`号

---
## 6.库操作
1. 创建数据库：
	- 基本语法：
		`Create database 数据库名字 [库选项]；`
		库选项：用来约束数据库，分为两个选项：
		--1.字符集设置：charset/character set 具体字符集（数据存储的编码格式），常用GBK,UTF8
		--2.校对集：collation 集体校对集（数据比较的规则）
		关于字符集和校对集的介绍[点这里](http://www.jb51.net/article/30865.htm)
		以下是标准写法：
				CREATE DATABASE test
				CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
		或者： 
				CREATE DATABASE test
				CHARSET 'utf8' COLLATE 'utf8_general_ci';
	- 注意：
	实测:一定要带上字符集与校对集一起，否则会报错，具体为什么报错我也不清楚。
	数据库的名字不能使用关键字或保留字，如果非要使用，那么必须用反引号:
			create database `database` charset 'utf8';
	中文名数据库是可以的，但是要要保证服务器能识别字符集，建议别用。
			create database 中国 charset 'utf8';
			create database `中国` charset 'utf8';
	都会出错。
	解决方案：在创建前提示字符集。（set names gbk）
			set names gbk;
			creat database 中国 charset 'utf8' ...;
	- 创建数据库的SQL语句执行之后发生了什么:
	1.在数据库系统中，增加了对应的数据库信息。
	2.会在保存数据的文件下：Data目录创建一个对应数据库名字的文件夹。但是中文数据库在此时会乱码，所以不推荐。
	3.每个数据库下都有一个opt文件，保存了库选项。（字符集，校对集）
2. 查看数据库：
	- 查看所有数据库：
			show databases;
	- 查看指定部分数据库：模糊查询
			show databases like 'pattern';
	pattern：匹配模式
	_：匹配单个字符
	%：匹配多个字符
	效果如下：
![](/img/00010mysql学习笔记一1.jpg)
	若有特殊字符则需要转义
	- 查看数据库的创建语句：
			show create database 数据库名字；
	若出现错误，检查一下是否是拼写错误。
3. 更新数据库（修改）：
**数据库名字：不可修改**（不安全）
数据库修改仅限库选项和字符集。
		Alter database dbname [库选项]；
		Charset/character set [=] 字符集
		Collate [=] 校对集
修改数据库`test`字符集为GBK：
		Alter database test charset GBK;
修改后校对集也会跟着改变。
一旦数据库里有数据后`最好不要修改`。指定为UTF8就行。
4. 删除数据库（所有操作的删除最简单）
结构的操作属于DDL：
		dorp database dbname;
删除test02数据库：
		drop database test02;
执行数据库删除后会发生什么：
--看不到数据库文件了
--数据库内部表全部删除
不要随意删除，需先备份后操作，删除不可逆。

---
## 7.表操作:
表与字段是密不可分的。
1. 新增数据表
		Create table [if not exists] 表名（字段名称 数据类型，字段名称 数据类型...）[表选项]
if not exists：如果表不存在则执行，否则不执行后面代码。
[表选项]：控制表的表现
&nbsp;&nbsp;&nbsp;&nbsp;-字符集:charset/character set 具体字符集;  --保证表中数据存储的字符集
&nbsp;&nbsp;&nbsp;&nbsp;-校对集:collate 具体校对集;
&nbsp;&nbsp;&nbsp;&nbsp;-存储引擎：innodb和myisam等;
<br>
例：创建一张学生表
若直接使用：
		create table if not exits student(
		name varchar(10),
		number varchar(10),
		age int
		)charset utf8;
会报错：原因是没有指定数据库，有两个解决办法：
1.显式地指定表所属数据库(在数据库环境外):
		create table [if not exists] dbname.tablename(...)[字符集]
2.隐式地指定表所属数据库(进入数据库环境，则表自动归属):
进入数据库环境：
		use dbname
若进入test数据库则使用`use test`。
再执行上述代码。
![](/img/00010mysql学习笔记一3.jpg)
创建数据表的SQL指令执行后发生了什么:
--指定数据库上已经存在的对应表
--在数据库对应的文件夹下会产生对应表的结构（跟存储引擎有关）:编译后的文件，需要反编译才能看
2. 查看数据表：
查看数据表之前需先进入某数据库环境，再用show查询:
		use dbname
有几种方式：
	- 查看所有表:
			show tables;
	- 查看部分表:模糊查询
			show tables 'parttern';
	如查询以s结尾的表:
			SHOW TABLES LIKE '%s';
	但这样会使索引失效，影响效率。
	- 查看表创建语句：
			show create table student\g  --\g效果和‘；’相同
			show create table student\G  --\G将查到的结果旋转90度，纵向
	- 查看表结构(三种方式):
			DESC student;
			descripe student;
			show colums from student;
	![](/img/00010mysql学习笔记一4.jpg)
3. 修改数据表：
包括修改表本身及修改字段。
	- 修改表本身：表名和表选项
	修改表名：
			rename table 老表名 to 新表名；
	修改表选项：字符集，校对集和存储引擎
			Alter table 表名 表选项 [=] 值；
	
	
---



