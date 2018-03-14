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
![](http://p5ki4lhmo.bkt.clouddn.com/00009mysql%E5%AD%A6%E4%B9%A01-1.jpg)
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
![](http://p5ki4lhmo.bkt.clouddn.com/00009mysql%E5%AD%A6%E4%B9%A01-2.jpg)
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
![](http://p5ki4lhmo.bkt.clouddn.com/00009mysql%E5%AD%A6%E4%B9%A01-3.jpg)
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
	![](http://p5ki4lhmo.bkt.clouddn.com/00009mysql%E5%AD%A6%E4%B9%A01-4.jpg)
3. 修改数据表：
包括修改表本身及修改字段。
	- 修改表本身：表名和表选项
	修改表名：
			rename table 老表名 to 新表名；
	修改表选项：字符集，校对集和存储引擎
			Alter table 表名 表选项 [=] 值；
	- 修改字段：增加，修改，重命名，删除
	1.新增字段：
			alter table 表名 add [column] 字段名 数据类型 [属性][位置]
	属性：是否自增，是否为空，是否有默认值等
	位置：字段可以存放在表中的任意位置
	--First:第一个位置
	--After column：在哪个字段后（后跟字段名）
	给学生表新增一个ID字段放到第一个位置：
			ALTER TABLE te_student
			ADD COLUMN id int
			FIRST;
	在name属性后添加一个sex性别:
			ALTER TABLE te_student ADD COLUMN sex VARCHAR(4) AFTER name;
	2.修改字段：位置，属性数据类型等:
			alter table 表名 modify 字段名 数据类型 [属性][位置]
	如修改字段age位置：
			ALTER TABLE te_student MODIFY age INT AFTER name;
	3.修改字段名字（也可以修改位置不加则不变）
			alter table 表名 change 旧字段 新字段名 数据类型 [属性][位置]；
	如修改te_student表中的name为stu_name:
			ALTER TABLE te_student CHANGE name stu_name VARCHAR(10);
	4.删除字段:
			alter table 表名 drop 字段名;
	注意:
	如果表中已经存在数据，会删除表中所有数据且不可逆。
4. 删除数据表:
		drop table 表名;
如删除st_student表：
		DROP TABLE te_student;

---
## 8.数据操作:
1. 新增数据:
有两种方案:
	1. 全表字段插入数据，不需要指定字段列表：**要求数据的值出现的顺序必须与表中设计的字段的顺序一致，凡是非数值数据都需要用引号（单引号）包裹**。
			insert into 表名 values(值列表)[,(值列表)]；  --可以一次性插入多条
	如给te_student添加我的信息（先把number数据类型改为varchar(20)）:
			INSERT INTO te_student VALUES(1,'kenshine','男',22,'201503111073');
	但是添加中文时要注意。参考第9条。
	2. 给部分字段插入数据，需要选定字段名列表：**子段列表的顺序与字段的顺序无关，但是致列表的顺序必须和字段列表顺序一致。**
			insert into 表名（字段列表） values(值列表)；
	如再给te_student添加一个Tom的信息:
			INSERT INTO te_student(id,name,number,sex,age) VALUES(2,'Tom','123456789','女',18);
	注意：非数值类型一定要加上单引号。
2. 查看数据(查询):
	- 简单查看:
			select * from 表名;
			SELECT * FROM te_student;
	- 查看指定字段，指定条件的数据:
			select 字段名[,字段名] from 表名 where 字段名 = 值；
	如查看我的学号和名字:
			SELECT number,name FROM te_student WHERE id=1;
3. 更新数据:
		Update 表名 set 字段=值[,字段=值] [where 字段名=值]；
建议都加上where,否则都会更新。
如更新学生Tom的性别为男：
		UPDATE te_student SET sex='男' WHERE name='Tom';
4. 删除数据:
删除是不可逆的，三思而后行。
		delete from 表名 [where 字段名=值]；    --建议加上where
把Tom删除:
		DELETE FROM te_student WHERE name='Tom';

---
## 9.中文数据的问题（在cmd中）
1. 原因:
计算机只识别二进制，人类更多的是识别符号，需要有个二进制与字符的对应关系（字符集）。
2. 存中文数据，会出现问题:
中文数据转换成了像`\xD5\xC5\xD4\xBD`这种形式,报错。
原因1:这是中文在当前(编码)字符集下对应的二进制编码转换为十六进制，两个汉字->四个字节-（GBK）。
原因2:服务器任务数据时UTF-8，一个汉字三个字节，第四个就出错了。
<br>
所有的数据库服务器的一些特性都是通过服务器端的变量来保存，系统会先读取自己的变量。
1.查看服务器能识别哪些字符集:
		show character set;
我的mysql查询出了41种，基本是都能识别的。
2.查看服务器与客户端交互的字符集(默认的对外处理的字符集):
		show variables like 'character_set%';
![](http://p5ki4lhmo.bkt.clouddn.com/00009mysql%E5%AD%A6%E4%B9%A01-5.jpg)
问题根源:第一行character_set_client,客户端对服务器的字符集为utf8,而客户端数据为gbk。
修改服务器对客户端的字符集为GBK:
		set character_set_client=gbk;
3. 上述问题解决之后，插入中文数据，使用select查看数据，仍然乱码：
原因:数据来源服务器，解析数据是客户端（服务器给三个字节UTF8，而客户端接收却是GBK）
解决方法:
		set character_set_result=gbk;
4. 但是，一个很严重的问题:
**set 变量=值；修改只是会话级别，当前客户端，当次连接有效，关闭失效**。
所以设置这种服务器的字符集有快捷方式:
		set names gbk;
它等价于：
`set character_set_result=gbk;`加`set character_set_client=gbk;`加`set character_set_connection=gbk;`
connection连接层，与client和result统一效率更高。

---
## 10.校对集:
1. 校对集：数据比较的方式。
2. 校对集有三种格式：
	- _bin (binary)：二进制比较，取出二进制位，从左到右一位一位的比较，区分大小写。
	- _cs (case sensitive)：大小写敏感，区分大小写（不常见）
	- _ci (case insensitive)：大小写不敏感，不区分大小写
3. 查看字符集:
查看服务器支持的所有字符集:
		show collaction;
4. 校对集的应用:
只有当数据产生比较时才会生效。如`order by字段名 [asc|desc]`才有效,asc升序，desc降序，默认是降序。
5. 注意：
校对集必须在没有数据之前设定好，如果有了数据再修改，那么修改无效。
具体如何修改字符集参考表的修改。
6. 老师是这么讲的，但是我的编码都是utf8在cmd下也没有乱码，很奇怪。
---
## 11.web乱码问题
1. 动态网站的构成：浏览器，Apache服务器（java后端），数据库服务器，三个部分都有自己的字符集（针对中文的），数据在三个部分来回传递，很容易产生乱码。
2. 最好的结果三码合一，但是很难做到。
解决方案:
java服务端--->浏览器(jsp自带):
		<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
浏览器--->服务器:表单由java服务端提供，也不用管。
<br>
java服务端--->数据库服务器端:`set character_set_client = utf8;`,
数据库服务器端--->java服务端:`set character_set_result = utf8;`,
一劳永逸的办法:`set names utf8;`
但是默认好像就是utf-8的。
<br>
数据库服务器<--->编码不同的数据库，数据库表：系统自动解决，不用管。
<br>
操作系统（GBK）--->java服务端字符集:
从本地读东西时，要改变自己字符集，如读取本地文件时:
		BufferedReader br=new BufferedReader(new InputStreamReader(new FileInputStream(fileName),"UTF-8"));

---
## 12.知识点补充
[MySQL的安装与配置，占位，待写](https://jingyan.baidu.com/article/cd4c2979033a17756f6e6047.html)


---





