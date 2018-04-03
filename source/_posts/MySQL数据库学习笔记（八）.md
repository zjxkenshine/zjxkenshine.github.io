---
title: MySQL学习笔记（八）：存储过程，游标，事件和字符集等
date: 2018-03-27 18:19:57
tags: MySQL
categories: 数据库

---
## 0.参考资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

---
## 1.存储过程的简单用法
**1)存储过程简介及创建：**
1. 存储过程简称为过程，procedure,是一种用来处理数据的方式。
**存储过程是没有返回值的函数。**各种操作都和函数类似（无返回值）。
2. 创建存储过程：
		Create procedure 过程名称([参数列表])
		Begin
			-- 过程体
		End
3. 创建测试：
		delimiter $$
		CREATE PROCEDURE pro1()
		BEGIN
			-- 过程中需要显式数据时需要用select
			SELECT * FROM m_class1;
		END $$
		delimiter ;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-01.jpg)

**2)存储过程的查看及调用：**
1. 函数的查看方式完全适用于过程，关键字改为perocedure。
查看过程：
		show procedure status [like '...']\G
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-02.jpg)
2. 查看过程创建语句：
		show create procedure 过程名；
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-03.jpg)
3. 调用存储过程：
过程没有返回值，说明不能用select访问。select调用时只会访问函数。
存储过程的调用(使用专门的关键字call)：
		call 过程名（[参数列表]）；
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-04.jpg)

**3)存储过程的删除：**
1. 存储过程一般不能修改内容，只能先删除后增加(可以修改特征值，属性)。
2. 删除语法：
		drop procedure 过程名；
测试:
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-05.jpg)

**4)存储过程参数：**
1. 函数的参数需要指定数据类型，过程比函数更加严格。
过程还有自己的类型限定：三种类型
	- in：数据只是从外部传入到内部使用(值传递)，可以是数据也可以是变量
	- Out：只允许过程内部使用（不用外部数据），给外部使用（引用传递：外部的数据会被清空后才进入到内部，存值后可在外部使用），只允许是变量。
	- InOut：外部的可以在内部使用，内部修改也可以在外部使用：典型的引用传递，只能穿变量。
2. 基本创建方法：
		create procedure 过程名（in 形参 数据类型,out 形参 数据类型,inout 形参 数据类型）;
		Begin
			-- 过程内容
		END
不代表in/out/inout三个都要，只是有的话必须这样写：`in 形参 数据类型`
测试创建：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-06.jpg)
3. 调用含参数的存储过程：
		call 过程名(参数列表);
**注意：out和inout传参时必须要传入一个变量，而不是数值**
使用如下代码测试：
		CALL pro2(1,2,3);		-- 未传变量
		set @int_1:=1;
		set @int_2:=2;
		set @int_3:=3;
		SELECT @int_1,@int_2,@int_3;
		CALL pro2(@int_1,@int_2,@int_3);
		SELECT @int_1,@int_2,@int_3;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-07.jpg)
4. out和inout是一种引用传递，内部修改一定会影响外部。
out的值传入过程内部时会被置空。

**5)存储过程对变量操作是滞后的：**
1. 滞后：**在存储过程结束的时候才会重新将内部修改的值传入给全局变量。**
2. 测试代码：
创建存储过程：
		delimiter $$
		CREATE PROCEDURE pro3(in int_1 int,out int_2 int,inout int_3 int)
		BEGIN
				-- 查看局部变量
				SELECT int_1,int_2,int_3;
				-- 修改局部变量
				SET int_1=100;
				SET int_2=1000;
				SET int_3=10000;
				-- 查看局部变量
				SELECT int_1,int_2,int_3;
				-- 查看全局变量
				SELECT @int_1,@int_2,@int_3;
				-- 修改全局变量(但是最后会被局部变量覆盖)
				SET @int_1='a';
				SET @int_2='b';
				SET @int_3='c';
				-- 再次查看全局变量
				SELECT @int_1,@int_2,@int_3;
		END
		$$
		delimiter ;
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-08-2.jpg)
测试代码：
		set @int_1=1;
		set @int_2=2;
		set @int_3=3;
		CALL pro3(@int_1,@int_2,@int_3);
		-- 存储过程结束后将out/inout的局部变量返回给全局变量
		SELECT @int_1,@int_2,@int_3; 
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-09-1.jpg)
3. 存储过程没有返回值但是同样可以将内部的结果返回给外部使用。

---
## 2.存储过程的补充
### 存储过程及函数的基础补充
**1)存储过程和函数基础知识补充(简单看一看就行)：**
1. 存储过程和函数是经过实现编译并存储在数据库中的SQL语句集合，调用存储过程可以简化应用开发人员的工作，减少数据在数据库和应用服务器之间的传输，有利于提高数据处理的效率。
2. 存储过程和函数的最大区别在于函数需要返回值而储存过程不需要，返回值。函数的参数类型只能为in类型，而存储过程的参数类型可以为in,out,inout三种类型。
3. 函数与存储过程中都允许包含DDL语句。
存储过程中能够执行啊事务的提交及回滚等。
存储过程和函数中决不允许执行LOAD DATA INFILE语句。
存储过程和函数中可以调用其他存储过程和函数。
4. **各种操作所需要的权限：**
创建存储过程和函数：需要Create Routine权限
修改存储过程和函数（特征值）：需要Alter Routine权限
执行存储过程：需要EXECUTE权限

**2)含特征值(属性)的函数/存储过程的创建及修改：**
1. 含特征值的函数/存储过程创建（以存储过程为例）：
		delimiter $$
		create procedure 过程名 （参数列表）
		[特征值列表]
		Begin
			-- 过程体
		End $$
		delimiter ;
2. 特征值列表的选项：
LANGUAGE SQL,[NOT] DETERMINISTIC,
{CONTAINS SQL|NO SQL|READS SQL DATA|MODIFILES SQL DATA}
|SQL SECURITY {DEFINER|INVOKER}|COMMENT 'string'
如果要指定多个属性选项，不要用逗号也不要用丨，只需要加空格就可以：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-10.jpg)
3. 修改存储过程或函数的属性，特征值。
		Alter procedure/function 过程/函数名 [特征值列表]
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-11.jpg)
4. 各特征值（属性简介）：
	- LANGUAGE SQL：系统默认的，说明函数/过程体内的语句是用SQL编写的。是为Mysql支持非SQL语言而准备的。
	- [NOT] DETERMINISTIC：
	`DETERMINISTIC`：确定的，每次输入一样输出也一样的程序。
	`NOT DETERMINISTIC`：默认，非确定的。
	- {CONTAINS SQL|NO SQL|READS SQL DATA|MODIFILES SQL DATA}：
	四个只能选择一个。提供子程序使用数据的内在信息，目前只是提供给服务器，没有根据这些值实际使用数据的情况：
		- `CONTAINS SQL`：默认，表示子程序不包含读或者写数据的语句。
		- `NO SQL`：子程序不包含SQL语句
		- `BEAD SQL DATA`：表示子程序包含读数据的语句但是不包含写数据的语句。
		- `MODIFILES SQL DATA`：表示子程序包含写数据的语句。
	- SQL SECURITY {DEFINER|INVOKER}：
	可以用来指定子程序用创建子程序者的权限来执行(difiner)，还是使用调用者的权限来执行（invoker）
	- COMMENT：存储过程或者函数的注释信息。
5. 特征值补充：
上述的子程序其实就是存储过程或者函数BEGIN~END包含的程序体。
DEFINER/INVOKER的权限表示存储过程内的语句权限与创建者/调用者的相同，如果创建者没有对表A的查询权限，而调用者有，则调用者调用SQL SECURITY DEFINER的存储过程时会报错说权限不够，而调用SQL SECURITY INVOKER的存储过程则不会报错。

### 定义条件和处理
**1)定义条件：**
1. 条件和处理可以用来定义在存储过程中遇到相应问题时的处理步骤。
2. 条件的定义语法：
		DECLARE 条件名 CONDITION FOR 条件值(condition_value);
3. 关于condition_value（条件值)可以填的数：
如一条错误出现，都是这种形式：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-12.jpg)
定义该错误的条件则condition_value处可以填：
`SQLSTATE '42000'` （SQLSTATE[VALUE] sqlstate值）
或者
`1064` (mysql错误值)

**2)定义条件的处理：**
1. 语法：
		DECLARE 处理选项 HANDLER FOR condition_value[...] 处理语句;
2. *处理选项*:
有三种可选值，但是只支持两种：undo不支持
`CONTINUE`:表示继续执行下面的语句；
`EXIT`:表示终止语句执行
3. condition_value[...] (条件值列表)：
有六种值：
	- `SQLSTATE[VALUE] sqlstate值`：和条件定义相同
	- mysql错误码，和条件定义相同
	- 条件名，使用条件定义语句定义的条件名
	- `SQLWARNING`：对所有以01开头的代码SQLSTATE代码的速记
	- `NOT FOUND`：对所有以02开头的代码SQLSTATE代码的速记
	- `SQLEXCEPTION`：对所有没有被`SQLWARNING`和`NOT FOUND`捕获的（不以以01或02开头的）SQLSTATE代码的速记
4. 处理语句：
出现指定错误时的处理语句。原本出错的那条语句不会生效改为指向这条处理语句。

**3)条件处理的测试**
1. 未设置条件处理的情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-13.jpg)
发现报错，根据该错误设置条件处理。
2. 设置了条件处理的情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-14.jpg)
出错语句后的语句正常执行。
3. 将条件处理器改为以下三种情况和上述条件处理是等价的。
		-- 使用sql错误码
		DECLARE CONTINUE HANDLER FOR 1062 SET @x2=666;
		-- 使用条件名
		DECLARE DuplicateKey CONDITION FOR 1062 ;
		DECLARE CONTINUE HANDLER FOR DuplicateKey SET @x2=666;
		-- 使用速记
		DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET @x2=666;

### 游标(光标)的使用
1. 光标(游标)简介：
也叫游标，在存储过程和函数中能使用光标对结果集进行循环处理。
是一种用于轻松处理多行数据的机制。
数据量非常大，则需要使用光标来逐条读取查询结果集中的记录 。
2. 光标的基本操作：
声明光标：
		DECLARE 光标 CURSOR FOR select语句;
打开光标：
		OPEN 光标名;
获取光标：
		FETCH 光标名 Into 变量1[,变量2...];
关闭光标：
		CLOSE 光标名;
如果没有明确的关闭光标，它会在其声明的复合语句的末尾被关闭。
3. 游标使用测试：
设计一个存储过程统计m_stu1表年龄大于25岁的人的身高总和：
		-- 光标测试
		delimiter $$
		CREATE PROCEDURE cur1()
		BEGIN
			-- 定义相关的变量（一定要和结果集字段区分开）
			DECLARE m_age TINYINT UNSIGNED;
			DECLARE m_height TINYINT UNSIGNED;
			-- 声明游标，加不加括号无所谓
			DECLARE cur_m_stu1 CURSOR FOR (SELECT age,height FROM m_stu1);
			-- 定义退出循环的条件处理
			DECLARE EXIT HANDLER FOR NOT FOUND CLOSE cur_m_stu1;
			-- 定义全局变量
			SET @sum=0;
			-- 打开光标
			OPEN cur_m_stu1;
			-- 死循环获取光标内的值
			REPEAT
				-- 获取光标内的一条数据并赋值给局部变量，并指向下一条，不重复取
				FETCH cur_m_stu1 INTO m_age,m_height;
				IF m_age >= 25 THEN
					SET @sum=@sum+m_height;
				END IF;
			-- 死循环条件，使用HANDLER条件处理来退出
			UNTIL 0 
			END REPEAT; 
			CLOSE cur_m_stu1;
		END $$
		delimiter ;
		
		CALL cur1();
		SELECT @sum;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-15-02.jpg)
4. 测试中总结游标使用时的注意事项：
FETCH获取游标给变量赋值时变量的数量，顺序（以及数据类型）一定要和声明时的select语句查询的结果集字段相同，否则会报错。
这是上述代码使用`select * from m_stu1`时的错误:
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-15.jpg)
还有一个低级错误，变量名不能和字段名相同，否则不报错，但是变量一直取不到值，这是我卡了好长时间的代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-15-01.jpg)


---
## 3.事件调度器(事件)的基本用法
**1)事件简介，创建及查看：**
1. 事件调度器是MySQL5.1后新增的功能，可以让数据库按照自定义的时间周期触发某种操作，可以理解为时间触发器。（书上只有基本用法，更加高大上的用法以后再学）
2. 简单的创建语法：
		CREATE EVENT 事件名字
		ON SCHEDULE EVERY 时间间隔 STARTS 开始时间
		DO
			要执行的SQL操作；
详细的创建语法：
	    CREATE  
	        [DEFINER = { user | CURRENT_USER }]  
	        EVENT  
	        [IF NOT EXISTS]  
	        event_name  
	        ON SCHEDULE schedule  
	        [ON COMPLETION [NOT] PRESERVE]  
	        [ENABLE | DISABLE | DISABLE ON SLAVE]  
	        [COMMENT 'comment']  
	        DO event_body;  
	      
	    schedule:  
	        AT timestamp [+ INTERVAL interval] ...  
	      | EVERY interval  
	        [STARTS timestamp [+ INTERVAL interval] ...]  
	        [ENDS timestamp [+ INTERVAL interval] ...]  
	      
	    interval:  
	        quantity {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |  
	                  WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |  
	                  DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}  
3. 测试：
创建一个m_stu2表：
		CREATE TABLE `m_stu2` (
		  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
		  `name` varchar(20) DEFAULT NULL COMMENT '学生姓名',
		  `create_time` datetime DEFAULT NULL,
		  PRIMARY KEY (`id`)
		) ENGINE=InnoDB DEFAULT CHARSET=utf8
创建事件调度器：
		-- 时间调度器
		CREATE EVENT add_stu 
		ON SCHEDULE EVERY 10 SECOND
		DO
		INSERT INTO m_stu2 VALUES(NULL,'test',NOW());
创建结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-16.jpg)
4. 如上图，查看事件创建语句：
		SHOW CREATE EVENT 事件名;
查看事件：
		SHOW EVENTS;
查看事件的全局设置（默认关闭）：
		SHOW VARIABLES LIKE '%scheduler%';
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-17.jpg)

**2)事件开启，关闭与删除：**
1. 事件环境：
事件全局环境是默认关闭的：`SHOW VARIABLES LIKE '%scheduler%'`查看，
所有的事件都是使用这一个变量。
可以使用如下代码开启环境：
		set global event_scheduler = on; -- 或者=1
关闭全局环境：
		set global event_scheduler = OFF; -- 或者=0
2. 事件开启以关闭
事件是默认开启的，上述的`status=ENABLED`。所以只要开启全局环境就可以运行事件了。如果发现是`disabled`关闭的可以这样开启：
		alter event 事件名 enable；
关闭事件：
		alter event 事件名 disable；
3. 事件开启后的进程：
事件不用主动使用，开启后就自动产生一个后台进程，可以使用：
		SHOW PROCESSLIST;
查看。
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-18-01.jpg)
使用`SHOW PROCESSLIST\G`查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-18-02.jpg)
4. 开启后隔几秒查看数据库，发现已经开始添加数据了，并且间隔为10秒：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-19.jpg)
5. 为了防止表变得很大，创建一个新的调度器，每隔一段清空一次表：
		CREATE EVENT tunc_stu
		ON SCHEDULE EVERY 10 MINUTE
		DO TRUNCATE TABLE m_stu2;
非常适合定期清空临时表或者日志表。
6. 事件的删除：
		drop event 事件名;
不想使用事件时可以禁用或者直接删除。

**3)事件调度器的优势，使用场景及注意事项：**
1. 优势：
MySQL事件调度器部署在数据库内部由DBA或专人统一维护和管理，避免将一些数据库相关的定时任务部署在操作系统层，减少操作系统管理员产生误差操作的风险，对后续的管理和维护也非常有益。
2. 适用场景：
适用于定期收集统计信息，定期清理历史数据，定期数据库检查等
3. 注意事项：
在繁忙且要求性能的数据库服务器上要慎重部署和启用调度器。
过于复杂的处理更适合用程序实现。
开启和关闭事件调度器需要超级管理员权限。

---
## 4.字符集与校对集
**1)字符集/校对集概述：**
1. 简单的说字符集就是一套文字符号及其编码、比较规则的集合。
2. 各种字符集的比较：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-20.jpg)
UTF8的汉字占三个字节。
3. Mysql支持多种字符集，一台服务器，一个数据库甚至同一个表不同字段都可以指定不同的字符集。
4. 查看所有可用的字符集：
		show character set;
或者使用：
		select * from information_schema.character_sets;
会显示出所有的字符集和改字符集默认的校对集。

**2)校对集和字符集的选择**
1. 校对集：
字符集（CHARACTER）用来定义MySQL存储字符串的方式。
校对集（COLLATION）用来定义字符串的比较方式。
字符集和校对集是一对多的关系。
可以使用：
		show collation like '%字符集%';
的方式查看。
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-21.jpg)
2. 校对集的比较：
通常由`字符集_语言名_结束标识`组成：
结尾是_cs：最常用，大小写不敏感
结尾是_ci：大小写敏感
结尾是_bin：大小写敏感，二元，比较的是编码，与语言无关
3. 关于utf8mb4字符集:
MySQL在5.5.3之后增加了这个utf8mb4的编码,是utf8的超集（GB18030是GBK的超集），mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。常用于存储一些UTF8三个字节无法存储的数据：包括 Emoji 表情（Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上），和很多不常用的汉字，以及任何新增的 Unicode 字符等等。一般为了节省空间使用utf8编码就已经足够了。
更多的区别可以参考：
[全面了解mysql中utf8和utf8mb4的区别](http://www.jb51.net/article/90037.htm)
4. 如何选取合适的字符集：
	- 满足地区语言的需求，建议使用utf8;
	- 如果涉及到已有数据的导入，要充分考虑对已有数据的兼容;
	- 数据库只需要支持一般中文，数据量大，性能要求也高，那么久应该选取双字节编码的GBK，而不是三字节编码的utf8;
	- 如果数据库需要大量的字符运算，如排序比较等，那么选择定长字符集可能更好。
	- 如果所有的客户端程序都支持相同的字符集，那么最好使用该字符集。

**3)字符集、校对集的设置及修改：**
MySQL的字符集和校对集有4个级别的默认设置：服务器级，数据库级，表级和字段级。
1. 服务器字符集和校对集：
可以在my.conf中设置：
		[mysqld]
		character-set-server=gbk;
或者在启动选项中指定(推荐)：
		mysql --character-set-sever=gbk;
或者在编译时指定：
		shell>cmake . -DDEFAULT_CHARSET=gbk;
校对集则会使用字符集默认的校对集。也可以手动指定。
查看当前字符集：
		show variables like 'character_set_server';
查看当前校对集：
		show variables like 'collation_server';
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-22.jpg)
2. 数据库字符集和校对规则：
可以在创建数据库时创建：
		CREATE DATABASE test
		CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
也可以在创建后修改：
		Alter database dbname [库选项]；
		 Charset/character set [=] 字符集
		 Collate [=] 校对集
如果创建时未指定字符集和校对集，则使用服务器的字符集和校对集。
如果只指定了字符集或者校对集其中之一，则会自动使用默认的字符集或校对集。
查看数据库的字符集（在数据库环境下使用）：
		show variables like 'character_set_database';
查看数据库的校对集：
		show variables like 'collation_database';
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-23.jpg)
3. 数据表的字符集和校对集：
可以在创建表的时候指定：
		Create table [if not exists] 表名（字段名称 数据类型，字段名称 数据类型...）ENGINE=.. CHARSET=.. COLLATE=..;
也可以通过修改表设置：
		Alter table 表名 表选项 [=] 值；
如果创建时未指定字符集和校对集，则使用数据库的字符集和校对集。
如果只指定了字符集或者校对集其中之一，则会自动使用默认的字符集或校对集。
显示表的字符集与校对集可以查看建表语句得知：
		show create table 表名；
4. 字段字符集和校对规则：
一般不太使用，在建表时设置字段的字符集，修改字段时也可以设置，只是为了更加灵活，一般情况下不会用到。
5. 连接字符集和校对集：
服务器和客户端之间交互的字符集和校对集的设置：
		set names 字符集;
详情见:
[MySQL学习笔记一中关于web乱码的部分](https://zjxkenshine.github.io/2018/03/10/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E4%B8%80%EF%BC%89/#11-web乱码问题)

---
