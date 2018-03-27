---
title: MySQL学习笔记（七）：变量、触发器、函数及流程控制
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
		End while [循环名];
4. 循环不适合在触发器中使用。
配合函数食用效果更佳
所以等学完函数再补充

---
## 4.函数及系统函数
**1)函数简介:**
1. 函数：将一段代码封装到一个结构中，需要执行代码块的时候调用结构执行即可。（代码复用）
所有函数都不区分大小写，系统函数最好用大写。
2. 函数分为两类：
系统函数，自定义函数
3. 系统函数：
系统定义好的函数，直接调用即可。
任何函数都必须要有返回值，所以可以通过select来查看调用。

**2)字符串系统函数：**
1. 常用的字符串系统函数：
	- concat(S1,S2...Sn)：连接S1到Sn为一个字符串
	- insert(str,x,y,instr)：将字符串str从第x位置开始，y个字符长的字符串替换为instr
	- lower/upper(str)：将字符串str变为小写/大写。
	- left/right(str,x)：返回字符串str最左/右边x个字符。
	- lpad/rpad(str,n,pad)：用字符串pad对str的最左边/最右边进行填充，直到字符串长度为n。
	- ltrim/rtrim(str)：去掉字符串最左边/右边的空格。
	- trim(str)：去掉行头和行尾的空格
	- repeat(str,x)：返回str重复x次的结果。
	- replace(str,a,b)：将str中所有的字符串a用b代替。
	- strcmp(a,b)：比较字符串a和b。
	- substring(str,x,y)：返回从x位置起y个字符长度的字符串。
	- char_length/length(str)：返回字符串长度。
2. 测试第一组方法：
		SELECT CONCAT('aaa','bbb'),CONCAT('aaa',NULL),INSERT('shabi',4,2,'diao');
		SELECT LOWER('HAHA'),UPPER('haha'),UPPER(NULL);
		SELECT LEFT('12345678',3),LEFT('12345678',NULL),RIGHT('12345678',3);
测试结果（注意为null的情况）：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-14.jpg)
3. 测试第二组方法：
		SELECT LPAD('123',10,'6'),LPAD('123',10,NULL),RPAD('123',10,'6');
		SELECT LTRIM('      haha'),RTRIM('haha     ');
		SELECT REPEAT('a',5),REPLACE('666666','6','7');
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-15.jpg)
4. 测试第三组方法：
		SELECT STRCMP('a','b'),STRCMP('a',null),STRCMP('a','a'),STRCMP('b','a');
		SELECT SUBSTRING('12345678910',3,5);
		SELECT LENGTH('123456'),CHAR_LENGTH('123456');
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-16.jpg)
注意：sql的下标是从1开始的。

**3)数值系统函数：**
1. 常用的数值函数：
	- ABS(x)：返回x的绝对值。
	- FLOOR/CEIL(x)：向下取整。
	- MOD(x,y)：返回x/y的值。
	- RAND()：返回0-1之内的随机值。
	- ROUND(x,y)：返回x四舍五入后带有y位小数的值。
	- truncate(x,y)：返回数字x截断为y位小数的结果。
2. 函数测试1：
		SELECT ABS(-0.8),ABS(0.8);
		SELECT CEIL(-0.8),CEIL(0.8),FLOOR(-0.8),FLOOR(0.8);
		SELECT RAND(),FLOOR(RAND()),CEIL(RAND());
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-17.jpg)
可以通过Rand()取到任意范围内的随机数。
3. 函数测试2：
		SELECT MOD(1,3),MOD(0,3),MOD(3,0),MOD(null,3),MOD(3,NULL);
		SELECT ROUND(1,2),ROUND(1.6,2),ROUND(1.66666,2),ROUND(1.4);
		SELECT TRUNCATE(1.66666,2),TRUNCATE(1.66666,0);
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-18.jpg)
MOD相当于%，任何人一个参数为null返回值都为null;
truncate和round的最大区别就是一个会四舍五入一个人只是截断。

**4)时间与日期函数：**
1. 常用日期与时间函数：
	- curdate()/curtime()：返回当前日期/时间
	- now()：返回当前的日期和时间
	- unix_timestamp(date)：返回日期date的unix时间戳
	- from_unixtime(...）：返回unix时间戳的日期值
	- week(date)：返回date是一年中的第几周
	- year(date)：返回date的年份
	- monthName(date)：返回date的月份名
	- hour(time)/minute(time)：返回time的小时值/分钟值
	- date_format(date,fmt)：按字符串fmt格式化日期date值
	- datediff(end,begin)：返回起始时间begin和结束时间end之间的天数
	- date_add(date,INTERVAL expr type)：返回与所给时间date相差INTERVAL时间段的值
2. 函数测试1：
		SELECT CURDATE(),CURTIME(),NOW(),WEEK(NOW());
		SELECT unix_timestamp(NOW()),from_unixtime(unix_timestamp(NOW()));
		SELECT YEAR(NOW()),MONTHNAME(NOW()),HOUR(NOW()),MINUTE(NOW());
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-19.jpg)
3. 最后三个函数介绍：
`date_format(date,fmt)`时间格式化函数：
mysql中的日期和时间格式非常多，主要有如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-20.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-20-0.jpg)
`date_add(date,INTERVAL expr type)`：返回与所给时间date相差INTERVAL时间段的值
如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-21.jpg)
如返回1年零两个月后的时间：
		SELECT DATE_ADD(NOW(),INTERVAL 1_2 YEAR_MONTH);
`datediff(expr,expr2)`：返回起始时间expr2和结束时间expr之间的天数
4. 测试最后三个函数：
		SELECT DATE_FORMAT(NOW(),'%D,%T,%Y,%M');
		SELECT DATE_ADD(NOW(),INTERVAL '1_2' YEAR_MONTH) 1year2monthlater;
		SELECT DATEDIFF(NOW(),DATE_ADD(NOW(),INTERVAL 60 DAY)) later2now,DATEDIFF(NOW(),DATE_ADD(NOW(),INTERVAL -60 DAY)) before2now;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-22.jpg)

**5)流程(分支)函数(和流程控制的稍有不同)**
1. 在一个SQL语句中实现条件选择可以提高语句的效率。
2. 常见的流程函数：
	- if(condition,t,f)：如果condition成立返回t,不成立返回f
	- ifnull(value1,value2)：如果value1不为空返回value1否则返回value2
	- `CASE WHEN value1 THEN [result1][when(value2)...] ELSE [default] end`
	满足条件value则返回result否则返回default。
	- `CASE [expr] WHEN value1 THEN [result1][when(value2)...] ELSE [default] end`
	满足条件value=expr则返回result,否则返回default。
3. 测试(m_stu2表)：
		SELECT id,name,if(Height>175,'tall','short') FROM m_stu1; -- 返回身高175以上为高，以下为矮
		SELECT id,name,(CASE height WHEN 161 THEN '残疾' WHEN 182 THEN '完美' ELSE '还行' END) as pingji FROM m_stu1;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-23.jpg)

**6)其他函数（加密函数等）**
1. 其他常用的函数：
	- DATABASE()：返回当前的数据库名
	- VERSION()：返回当前的数据库版本
	- USER()：返回当前登录用户
	- INET_ATON(IP)：返回IP地址的数字表示
	- INET_NTOA(num)：返回数字表示的IP地址
	- PASSWORD(str)：返回字符串str的加密版本，不能用来对应用数据加密
	- MD5(str)：返回str的MD5加密版本，常用于应用的数据加密
2. 函数测试：
		SELECT DATABASE(),VERSION(),USER();
		SELECT INET_ATON('192.168.1.1'),INET_NTOA('3232235777');
		SELECT PASSWORD('123456'),MD5('123456');
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-24.jpg)
3. PASSWORD与MD5的简单区别：
password用于修改mysql的用户密码，如果是应用与web程序建议使用md5()函数
password函数旧版16位，新版41位，可用`select length(password('123456'))`察看。
password函数加密不可逆，如果和数据库里加密后内容比较时可以采用`password(pwd)==字段内容`的方式；
md5函数加密后32位，此加密算法不可逆，其实md5算法是信息摘要算法，如果拿来做压缩也是有损压缩，理论上即使有反向算法也无法恢复信息原样。常被用来检验下载数据的完整性。

---
## 5.自定义函数及函数的操作
**1)自定义函数：**
1. 函数的要素：
函数名，参数列表(形参，实参)，返回值，函数体(作用域)。
2. 自定义函数与系统函数调用的方式相同。

**2)自定义函数创建与查看：**
1. 创建语法：
		create function 函数名([形参列表]) returns 数据类型 
		Begin 
				-- 函数体
				-- 返回值：return的类型（指定的数据类型）
				-- 只有一行时不用begin和end
		End 结束表标识
注意设置返回值时是returns。
2. 创建测试：
		CREATE FUNCTION display1() RETURNS INT return 100;
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-25.jpg)
3. 查看所有自定义函数：
		show function status[like '...'];
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-26.jpg)
发现该数据库下共有23个函数,第一行指定了数据库，说明只有这个数据库能使用该函数。
4. 查看函数的创建语句：
		show create function 函数名；
如：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-27.jpg)

**3)函数删除及带参数的函数的创建：**
1. 函数无法修改，只能先删除，后新增：
		drop function 函数名;
2. 函数的参数：
参数分为两种，定义时的参数--形参，使用时的参数--实参。
形参：必须指定数据类型。
3. 带参数的函数的创建：
		delimiter $$
		create function 函数名(形参名字 字段类型[...]) returns 数据类型 
		Begin 
				-- 函数体
				-- 返回值：return的类型（指定的数据类型）
				-- 只有一行时不用begin和end
		End $$
		delimiter ;
如果函数体中需要使用`；`做结束标志，则需要使用`delimater`修改结束符。
4. 创建测试：
创建一个sum1函数求1到n的和：
-- 自定义求和函数
		delimiter $$ 
		CREATE FUNCTION sum1(n INT) RETURNS INT
		BEGIN
				-- 定义条件变量
				SET @sum=0;
				SET @i=1;
				-- 循环求和
				WHILE @i<=n DO
						-- 求和,mysql中任何变量的修改都必须用set关键字
						-- mysql中没有+=，++等运算符
						SET @sum =@sum+@i;
						-- 修改循环变量
						SET @i=@i+1;
						-- 返回值
				END WHILE;
				RETURN @sum;
		END
		$$
		delimiter ; 
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-28-2.jpg)
带有@的变量时全局变量，不带@的函数是局部变量。
5. 使用上述函数进行测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-29.jpg)

**4)作用域：**
1. 在函数内部使用@定义的符号，在外部也可以使用。
全局变量能在任何地方使用，局部变量只能在函数内部(begin~end间)使用。
2. 全局变量：
使用set关键字定义，并使用@符号标识：
		set @变量名=值；
3. 局部变量：
使用declare关键字声明，没有@符号标识。且必须在函数体开始之前声明：
		declare 变量名1[,变量名2...] 类型 default 默认值；
修改局部变量的值：
		set 变量名=值；
也可以通过查询直接赋值：与全局变量相同。
4. 测试局部变量：
创建一个sum1函数求1到n的和，但是5的倍数不加：
		delimiter $$ 
		CREATE FUNCTION sum2(n INT) RETURNS INT
		BEGIN
				-- 声明局部变量 
				DECLARE sum INT DEFAULT 0;
				DECLARE i INT DEFAULT 1;
				-- 循环求和
				oneton:WHILE i<=n DO
						-- 判断如果是5的倍数跳出该次循环
						IF MOD(i,5)=0 THEN
								SET i=i+1;
								ITERATE oneton;
						END IF;
						-- 求和
						SET sum =sum+i;
							-- 修改循环变量
						SET i=i+1;
				END WHILE;
				-- 返回值
				RETURN sum;
		END
		$$
		delimiter ; 
创建及测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00022mysql%E5%AD%A6%E4%B9%A07-30.jpg)

---
## 6.流程控制补充
除了if,while外mysql还有一些常用的流程控制语句
**1)CASE语句：**
1. case语句可以实现比if更复杂的条件结构
2. 基本语法：
		CASE case_value
			WHEN value1 THEN result1[when value2 then...]
			[ELSE result];
		END CASE;
或者：
		CASE 
			WHEN 条件1 THEN 结果1[when 条件2 then...]
			[ELSE result]
		END CASE;

**2)LOOP语句：**
1. 能实现简单的死循环。
2. 一般创建形式：
		LOOP
			-- 循环体：操作
		END LOOP;
3. 如果要退出死循环需要结合if和iterate/leave操作：
		循环名字：Loop
			-- 循环体
			-- 循环控制
			Iterate/Leave 循环名字
		End Loop [循环名];

**3)REPEAT语句：**
1. REPEAT:满足条件时退出循环，相当于`do while ... until`。
2. 基本语法：
		REPEAT
			-- 循环体：操作
		UNTIL 终止循环的条件
		END REPEAT;
3. 配合iterate/leave操作：
		循环名字：REPEAT
			-- 循环体
			-- 循环控制
			Iterate/Leave 循环名字
		UNTIL 终止循环的条件
		End REPEAT [循环名];
可以看出三种循环其实差不多。

---