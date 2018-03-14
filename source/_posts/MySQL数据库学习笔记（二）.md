---
title: MySQL学习笔记（二）:基本数据类型与列属性介绍
date: 2018-03-10 16:20:31
tags: MySQL
categories: 数据库

---
## 0.学习资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

---
## 1.关于数据类型（列类型）
1. 为什么要分类:
二维表即使不存数据，定义之后也会浪费空间。
2. 数据类型:对数据进行统一的分类，从系统的角度出发为了能够使用统一的方式进行管理，更好地利用有限空间。
3. 总共分为了三大类:数值型，日期处理类型，字符串型

---
## 2.数值型
存放的值为数值，系统将数值型分为了整数型和小数型。而内部又会细分。
数值类型简单介绍表：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-1.jpg)
### 2.1 整数型
存放整型数据：在SQL中因为更多要考虑如何节省磁盘空间，所以系统又将整型细分为了5类：
1. 整型的分类
	- Tinyint:迷你整型，使用1个字节存储，表示的状态最多为256（2^8）种（常用）
	- Smallint:小整型，使用2个字节存储，表示的状态最多为65536（2^16）种
	- Mediumint:中整型，使用3个字节存储,表示的状态最多为2^24种
	- int:标准整型，使用4个字节存储（32位），表示的状态最多为2^32种(常用)
	- Bigint:大整型，使用8个字节存储（64位），表示的最多的有2^64种
2. 整型的测试:
创建整型表：
		USE test
		CREATE TABLE IF NOT EXISTS te_int(
		int_1 TINYINT,
		int_2 SMALLINT,
		int_3 MEDIUMINT,
		int_4 INT,
		int_5 BIGINT
		)CHARSET utf8;
只能插入整型且只能插入范围内的数据:
		INSERT INTO te_int values(100,100,100,100,100);   -- 有效数据
		INSERT INTO te_int values('a','b',100,'c',100);  -- 无效数据，类型不正确
		INSERT INTO te_int values(255,65530,10000,10000,10000);  -- 无效数据，超出范围
tinyint值存255会超出范围，原因如下:
SQL中的数值类型,分正负，有符号的tinyint的范围是-128~127，可以设置为无符号型，范围为0-255，无符号类型使用`unsigned`属性设置:
		ALTER TABLE te_int ADD COLUMN int_6 TINYINT UNSIGNED AFTER int_5;  -- 往int_6中添加255则不报错
3. 显示宽度:
添加无符号tinyint后，使用desc查看各个整型的类型:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-2.jpg)
括号后的数字代表了**显示位数**，如-123是4位，255是三位。
**显示宽度**:没有特别含义，用户是可以控制的，这种控制不会改变数据本身的大小。
		ALTER TABLE te_int ADD int_7 int(1) AFTER int_6;
		INSERT INTO te_int(int_7) VALUES(110);  -- 但是可以插入
**显示宽度的意义**：在于当数据不够显示宽度的时候，会自动让数据变成对应的显示宽度，通常需要搭配一个前导0来增加宽度，不改变值大小（`zerofill`属性0填充）
		ALTER TABLE te_int ADD int_8 tinyint(3) zerofill;
		INSERT INTO te_int(int_8) VALUES(2);
0填充会自动使tinyint变为无符号位。
效果：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-3.jpg)
但是在Navicat等管理工具中看不到0。
**零填充的意义**：保证数据格式（html显示）

### 2.2 小数型
1. 小数型：带有小数点或者范围超出整型的数值类型。
在SQL中将小数型又分为浮点型和定点型：
浮点型：小数点浮动，精度有限，而且会丢失精度。（四舍五入）
定点型：小数点固定，精度固定，不会丢失精度。
2. 浮点型:
	- 浮点型数据又称为精度型数据：超出指定范围之后，会丢失精度（自动四舍五入）。
	浮点型又分为两种:
	&nbsp;&nbsp;&nbsp;&nbsp;--Float:单精度，占用四4个字节(32位)存储，精度范围大概为7位左右。
	&nbsp;&nbsp;&nbsp;&nbsp;--Double：双精度，占用8个字节存储，精度范围大概为15位左右。
	- 创建浮点型数表：
	方式1：直接float，表示没有小数部分
	方式2：float（M，D）:M代表总长度，D代表小数长度，整数占M-D位。
			CREATE TABLE te_float(
			fl FLOAT,       -- 整数长度超过精度也会四舍五入
			f2 FLOAT(10,6),  -- 在float精度范围7之外
			f3 FLOAT(6,2),  -- 在float精度范围7之内
			d1 DOUBLE(10,6)
			)CHARSET utf8;
	可以创建成功，但是给该表插入数据时:
			INSERT INTO te_float VALUES(1000.50,1000.50,1000.50,1000.50);  -- 符合条件
			INSERT INTO te_float VALUES(1234567890,1234.567890,1234.56,1234.567890)  -- 符合条件
			INSERT INTO te_float VALUES(9.999999999e10,9999.999999,9999.99,9999.99) -- 符合条件最大值
	数据可以使用科学计数法形式。先看一下插入规则:
	**浮点数插入规则**:整数部分是不能超出长度的，但是小数部分可以超出长度（四舍五入）。
	下面是添加结果:
		![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-4.jpg)
	结论:超出精度范围的浮点数一定会进行四舍五入，浮点数如果是因为系统进位而导致整数部分长度超出指定范围，系统也是允许的。
3. 定点型(DECIMAL):
	- 绝对的保证**整数部分**不会被四舍五入（不会丢失精度），小数部分有很小的可能会丢失精度。
	DECIMAL（M，D）：编长，大致是每9个数字采用4个字节存储，整数与小数分开计算,
	M最大值是65，D最大值是30，默认值是（10，2）
	- 创建定点表:
			CREATE TABLE te_decimal(
			d1 DECIMAL(10,1),
			f1 FLOAT(10,1)
			)
			CHARSET utf8;
	插入数据:定点数的整数部分一定不能超出长度（不会进位），小数部分可以随意超出(系统自动四舍五入)：
			INSERT INTO te_decimal VALUES(12345678.90,12345678.90);
			INSERT INTO te_decimal VALUES(12.3434567890,12.3434567890);
	使用cmd插入时系统会报warning，可以使用`show warning`查看。
	浮点数如果进位导致长度增加没问题，但是定点数会出现问题:
			INSERT INTO te_decimal VALUES(999999999.9,999999999.9);      -- 有效数据
			INSERT INTO te_decimal VALUES(999999999.99,999999999.99);    -- 进位
	定点数会报错提示`out of range`。下面是添加结果:
	![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-5.jpg)
4. 浮点数与定点数的选择：
**要求精度低的使用浮点数，要求精度高的使用定点数。**

---
## 3.时间日期类型
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-6.jpg)
补充介绍:
DateTime：时间日期，格式是YYYY-mm-dd HH:MM:ss,有0值，0000-00-00 00:00:00。
Time：更多表示时间段，也可以表示时间，指定的某个区间之间。
Year：year(2),year(4),默认为4位。
1. 创建时间日期表:
		CREATE TABLE IF NOT EXISTS te_date(
		d1 datetime,
		d2 date,
		d3 time,
		d4 TIMESTAMP,
		d5 YEAR
		)CHARSET utf8;
使用desc查看表结构，可以看到时间戳的特殊性:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-7.jpg)
时间戳默认不为空，默认值为当前时间，插入时不明确timestamp赋值时也会使用当前时间，但是注意，timestamp只会反映当地时区的时间。
2. 插入数据：
注意：时间time可以使用负数（很大的负数），year可以使用两位插入。
		INSERT INTO te_date VALUES('2018-3-14 17:25:10','2018-3-14','2018-3-14 17:25:10','2018-3-14 17:25:10',2018);  -- 正常数据，time解析为17:25:10
		INSERT INTO te_date VALUES('2018-3-14 17:25:10','2018-3-14','-211:25:10','2018-3-14 17:25:10',2018);  -- time存负值，这种写法直接存入
		INSERT INTO te_date VALUES('2018-3-14 17:25:10','2018-3-14','-2 11:25:10','2018-3-14 17:25:10',2018);  -- 表示两天前，会自动转换为-59:25:10
		
		INSERT INTO te_date VALUES('2018-3-14 17:25:10','2018-3-14','11:25:10','2018-3-14 17:25:10',69);  -- year存两位，解析为2069年
		INSERT INTO te_date VALUES('2018-3-14 17:25:10','2018-3-14','11:25:10','2018-3-14 17:25:10',70;  -- year存两位，解析为1970年
timestamp特性:
只要它所在的某条记录被更改(非timestamp字段),timestamp也会自动修改更新为当前时间。在Navicat等可视化操作修改（不用update）timestamp不会更新。
可以使用`select unix_timestamp();`得到系统的时间戳(整型)。
注意:PHP的date只要一个时间戳（整型）就可以解析，一般不使用。
java的Date没研究过，最好还是用time类型吧。

---
## 4.字符串类型
在SQL中，将字符串类型分成了6类：**char,varchar,text,blog,enum和set**。
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-9.jpg)
1. 定长字符串：**char**
	磁盘（二维表）在定义结构的时候，就已经确定了最终数据的存储长度。
	Char(L)：L代表length,可以存储的长度，单位为字符，最大长度值可以为255。
	Char(4)：在UTF8环境下，需要4\*3=12个字符。
	<br>
变长字符串:**varchar**
	在分配空间是按照最大的空间分配，但实际上用了多少是根据具体的数据来确定的。
	Varchar(L)：L代表字符长度，理论长度是65536个字符，但是会多出1-2个字节来确定存储的实际长度（255个字以下1位，255个字以上2位）
	Varchar(10)：若存满了10个字实际使用的字节10\*3+1=31个字节。若只存储了3个字符则实际上使用了3\*3+1=10个字符。

2. 定长与变长的比较:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-8.jpg)
如何选择定长和变长字符串:
数据长度固定--定长：身份证，电话号码，学号等。
长度不固定--变长：姓名，住址等其他很多。
定长的磁盘空间比较浪费，但是效率高。
变长磁盘空间比较节省，但是效率低。

3. 文本字符串:
如果数据量非常大就会使用文本字符串（超过255个字符）：
文本字符串根据存储的数据的格式进行分类：text和blog
<blockquote>Text:存放文字。
Blog:存放二进制数据（通常不用，通常存路径）。</blockquote>

4. 枚举字符串（ENUM）
**枚举**:事先将所有可能出现的结果都设计好，实际上存储的数据必须是规定好的数据中的一个。（性别最常用）
使用方法：
		CREATE TABLE te_enum(
		gender enum('男','女','保密')
		)CHARSET utf8;
插入数据：
		INSERT INTO te_enum VALUES('男'),('女'),('男'),('保密');
		INSERT INTO te_enum VALUES('1');  --也可以直接插入数字
枚举作用一：规定数据格式，数据只能是规定的数据中的其中一个。
**在mysql中，系统也是自动转换数据格式的，尤其是字符转数字。**
枚举作用二：节省存储空间（别名，单选框），枚举实际存的是数值而不是字符串本身。
证明存的是数值：将结果取出并+0运算，若是0则是字符串，不是就是数值。
		select gender +0,gender from te_enum;
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-10.jpg)
枚举元素的实际规律：按照实际出现的顺序从1开始编号。只用（1或2个字节存储）
**枚举原理**:
枚举在进行数据规范的时候(定义的时候)，系统会自动建立一个数字与枚举元素的对应关系（存在日志中），然后在进行数据插入的时候，系统自动将字符串转换成对应的数字存储，然后再进行数据提取的时候，系统自动将数值转换为对应的字符串显示。

5. 集合字符串
集合和枚举类似，实际存储的是数值而不是字符串（集合是多选，且最多只能有64个选项）
<blockquote>
集合使用方式：
定义:Set(元素列表)
使用:可以使用元素列表中的一个或多个元素，用','分隔。
</blockquote>
创建集合表:
		CREATE TABLE te_set(
		hobby SET('篮球','足球','羽毛球','乒乓球','睡觉','敲代码')
		)CHARSET utf8;
插入数据:可以使用多个元素字符串组合，也可以插入数值
		INSERT INTO te_set VALUES('足球,睡觉,敲代码');
		INSERT INTO te_set VALUES('足球,羽毛球,足球');  -- 重复的只存一次
		INSERT INTO te_set VALUES(3);     -- 3=1+2 会存进篮球+足球，不会存3羽毛球
使用`select hobby +0,hobby from te_set;`可以查看原因:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-11.jpg)
50转换为二进制位：32+16+2=00110010
将set集合从零开始颠倒向右对齐（8，7，**6**，**5**，4，3，2，**1**，0）。
**集合中每一个元素都是对应着一个二进制位**。所以3=2^0+2^1。
集合的强大在于能够规范数据和节省空间。但是效率低。
6. Mysql记录的长度：
Mysql中规定:任何一条**记录**最长不能超过65535个字符（varchar永远不能达到理论值）
	- Varchard的实际存储长度,不同的字符集不同:
	<blockquote>
	UTF8：21844\*3+2=65534   -- 最多可以存储21844个字符
	GBK：32766\*2+2=65534   -- 最多可存储32766字节
	</blockquote>
	还多一个字节:
	Mysql记录中：**如果有任何一个字节允许为空，那么系统会从整个记录中保留一个字节来存储NULL**。
	- **关于text**：
	Mysql中的text文本字符串，不占用记录长度：额外存储，但是text文本字符串也是属于记录的一部分，一定要占用记录中的部分长度：**10个字节**（保存数据地址和长度）

---
## 5.列属性：
待学习

---
