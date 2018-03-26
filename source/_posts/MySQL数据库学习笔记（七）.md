---
title: MySQL学习笔记（七）：变量、触发器、函数、分支/循环及存储过程
date: 2018-03-25 18:51:00
tags: MySQL
categories: 数据库

---
## 0.学习准备
1. 学习资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

系统函数部分及补充部分参考书本
其余部分参考视频

2. 建表准备：创建订单表与货物表并添加数据
		-- 创建商品表
		CREATE TABLE m_goods(
		id INT PRIMARY KEY auto_increment,
		name VARCHAR(20) NOT NULL,
		price DECIMAL(10,2) DEFAULT 1,
		inv INT COMMENT '库存容量'
		)CHARSET utf8;
		-- 添加商品
		INSERT INTO m_goods VALUES(NULL,'iphone100',5288,100),(NULL,'iphone20',60000,200);
		
		-- 创建订单表
		CREATE TABLE m_order(
		id int PRIMARY KEY auto_increment,
		g_id int NOT NULL COMMENT '商品Id',
		g_number int COMMENT '商品数量'
		)CHARSET utf8;

---
## 1.变量
变量分为两种：
系统变量和自定义变量。
**1)系统变量：**
1. 系统定义好的变量，大部分的时候用户根本不需要改变系统变量。
系统变量时控制服务器的表现的：
如：autocommit,auto_increment,engine,字符集，校对集等
2. 查看系统变量:
	- 查看所有系统变量：
		Show variables;
发现有超级多的系统变量：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-01.jpg)
（警告可以使`show warings`查看）
	- 查看具体的变量值：
		select @@变量名； -- 加@@才能查看系统变量
任何一个有数据返回的内容都需要用select来查看。
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-02.jpg)
3. 修改系统变量:分为两种--会话级别和全局级别
-- 会话级别:当前客户端当次连接有效（两种方式）：
		set 变量名=值；
或者：
		set @@变量名=值；
-- 全局级别:一次修改，永久有效：
		set globle 变量名 = 值；
4. 如果其他客户端意见连上了服务器，那么当次修改无效。
需要退出重新登录才会生效。
5. 除非有需求要改变系统特征，否则一般情况不会使用系统变量。

**2)自定义变量：**
1. 定义变量：
系统为了区分系统变量和自定义变量，规定用户自定义变量必须使用一个@来赋值。
		set @变量名 = 值；
如：
		-- 定义自定义变量
		SET @name='kenshine';
2. 查看自定义变量名：
		select @变量名；
注意：如果不加@则会被当做是查字段。
		-- 查询自定义变量
		SELECT @name;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-03.jpg)
3. 注意：
mysql中的`=`经常被用来当做比较符号，所以mysql又重新定义了一个赋值符号：
`:=`
定义变量与查看变量可以这样使用：
		SET @name :='kenshine';
		SELECT @name;

**3)从数据表中获取数据赋值给变量：**
1. Mysql允许从数据表中或获取数据然后赋值给变量:有两种方式
2. 第一种方式：
变赋值边查看结果：
		Select @变量名 :=字段名[,字段名...] from 表名；
此时如果用`=`就会被解析成比较符号，所以一定得用`=`。
如：
	SELECT @name = name FROM m_stu1;
	SELECT @name := name FROM m_stu1;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-04.jpg)
但是这种方式赋值一次值获取最后一条记录，通常不使用这种方式赋值。
3. 第二种方案：
只有赋值，不会输出结果。
但是要求很严格：数据库或取数据只能获取一条，因为Mysql不支持数组。
		select 字段1[,字段2...] from 表名 [where...] into @变量名1[@变量名2...] 
可以同时给多个变量赋值，但是只能在查询出一条的情况下赋值，否则会报错:
		SELECT name,classid FROM m_stu1 WHERE id = 4 INTO @name,@classid;
		SELECT @name,@classid;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-05.jpg)
4. 注意：
**所有的自定义变量都是会话级别的。**
所有的自定义变量都不区分数据库。（用户级别）

---
## 2.触发器
使用情景：
有两张表，一张订单表，一张商品表，每生成一笔订单，商品库存需要对应减少。
**1)触发器简介：**
1. 简介：
trigger,事先为某张表绑定好一段代码，当表中的内容发生改变时（增删改），系统会自动触发器代码，执行。
2. 触发器的基本组成：**事件类型，触发时间，触发对象**
事件类型：三种类型，增删改，Insert,delete和update;
触发时间：前后，before和after;
触发对象：表中的每一条记录（针对行）。
3. **一张表最多拥有一种触发时间的一种触发类型的触发器，也就是一张表最多能有6个触发器。**

**2)创建触发器：**
1. 注意：
在mysql高级结构中没有大括号（的语法），所以创建触发器时不能用括号，只能用Beign和End代替括号。
2. 创建语法：
		-- 临时修改语句结束符
		Delimiter 自定义结束符（假设为$$）
		Create trigger 触发器名字 触发时间 事件类型 on 表名 for each row Begin 
			-- 触发器事件（内容），每行都以;结束。
		End 自定义结束符($$)
		-- 将语句结束符改回；
		Delimiter ;
3. 给订单表创建触发器：
		delimiter $$
		CREATE TRIGGER aftig AFTER INSERT ON m_order FOR EACH ROW 
		BEGIN 
			-- 触发器内容(有问题需要改进)
			UPDATE m_goods SET inv=inv-1 WHERE id=2;
		END 
		$$
		delimiter ;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-06.jpg)

**3)查看触发器：**
1. 两种方式：查看所有触发器或者模糊匹配
		Show triggers [like '...'];
2. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-07.jpg)
可以使用\G参数使结构变的清楚一点：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-08.jpg)
3. 查看触发器创建语句：
		show create trigger 触发器名字；
如：
		SHOW CREATE TRIGGER aftig;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-09.jpg)
4. 所有的触发器都被保存在一张表中：`Information_schema.triggers`
所以可以通过查询该系统表查看触发器:
		SELECT * FROM Information_schema.triggers;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-10-1.jpg)

**4)使用触发器：**
1. 不需要手动调用，而是某种情况发生时会自动调用
2. 上述的触发器内容是：
		UPDATE m_goods SET inv=inv-1 WHERE id=2;
当订单购买了两个商品1时：
		INSERT INTO m_order values(NULL,1,2);
执行并查询商品表：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-11.jpg)
发现商品二被修改，且只减少了一个。但是触发器的确工作了。
说明触发器并不合理；（后面会解决）
3. 千万不要在触发器内容中修改同一个表，否则会陷入死循环。

**5)删除触发器：**
1. 触发器不能修改，只能先删除后新增。
2. 删除触发器的语法：
		drop trigger 触发器名字；
测试：
		DROP TRIGGER aftig;
此时再添加订单不会有商品减少。

**6）上述问题的解决:触发器记录**
1. 触发器记录：
不管触发器是否触发了，只要某种操作准备执行，系统就会将当前要操作的记录的当前状态和即将执行后的新状态分别保留下来供触发器使用，其中当前状态保存在old中，操作之后的可能形态保存给new.(触发器查看中有)
2. old代表的是旧记录，new代表的是新记录：
--删除的时候是没有new的，插入的时候是没有old的。
3. 使用方式：
		old.字段名  /  new.字段名;
4. 修改触发器内容：
		Begin
			UPDATE m_goods SET inv=inv-new.g_number WHERE id=new.g_id;
		End
5. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-12.jpg)
能正确处理。

---
## 3.代码执行结构
顺序结构，分支结构，循环结构
**1)分支：if**
1. 按照条件选择性执行某段代码：
在mysql中只有if分支。
2. 分支语法：
		if 条件判断 then
			-- 满足条件的代码
		else
			-- 不满足条件的代码
		end if;
3. 触发器结合分支：
商品库存够：生成订单；不够：不生成订单；
		delimiter $$
		CREATE TRIGGER beftig BEFORE INSERT ON m_order FOR EACH ROW
		BEGIN 
				-- 判断库存是否足够
				-- 获取商品库存
				SELECT inv FROM m_goods WHERE id=new.g_id INTO @inv;
				-- 比较库存
				if @inv<new.g_number THEN
				-- 库存不足：mysql没有提供阻止触发器事件发生的方法，只能暴力停止
						INSERT INTO xxxx VALUES (xxxx);
				end IF;
		END 
		$$
		CREATE TRIGGER aftig AFTER INSERT ON m_order FOR EACH ROW 
		BEGIN 
					-- 触发器内容
					UPDATE m_goods SET inv=inv-new.g_number WHERE id=new.g_id;
		END 
		$$
		delimiter ;
测试：
		Insert into m_order values(null,1,100000);
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-13-1.jpg)

**2)循环结构：while**
1. 代码在指定条件之下重复循环：常用while语法
2. 语法：
		While 条件判断 do
			-- 满足条件要执行的代码
			-- 变更循环条件
		End while;
3. 循环控制：
在循环内部进行循环判断和控制
Mysql中没有continue和break，但是有相似的;
Iterate:迭代，类似continue
Leave：离开，类似break，整个循环结束 
用法：
		Iterate/Leave 循环名字
所以需要定义循环名字：
		循环名字：While 条件判断 do
			-- 循环体
			-- 循环控制
			Iterate/Leave 循环名字
		End while;
4. 循环不适合在触发器中使用。

---
## 4.代码执行结构补充


---
## 5.函数及系统函数
**1)函数简介:**
1. 函数：将一段代码封装到一个结构中，需要执行代码块的时候调用结构执行即可。（代码复用）
2. 函数分为两类：
系统函数，自定义函数
3. 系统函数：
系统定义好的函数，直接调用即可。
任何函数都必须要有返回值，所以可以通过select来查看调用。

**2)字符串系统函数：**


---
## 6.自定义函数及函数的操作

---
## 7.存储过程的使用

---
## 8.存储过程补充


---