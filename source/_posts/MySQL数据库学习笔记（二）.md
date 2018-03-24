---
title: MySQL学习笔记（二）:基本数据类型与列属性介绍
date: 2018-03-12 16:20:31
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
	- 绝对的保证**整数部分**不会被四舍五入（不会丢失精度），小数部分有很小的可能会丢失精度。以字符串的形式存储。
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
在SQL中，将字符串类型分成了6类：**char,varchar,text,blob,enum和set**。
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
Blob:存放二进制数据（通常不用，通常存路径）。</blockquote>

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
### 空属性，注释，默认值
列属性：真正约束字段的是数据类型，但是数据类型的约束很单一，需要有一些额外的约束来更加保证数据的合法性。
列属性简介：NULL/NOT NULL,default,Primary key,unique key,auto_increment,comment
1. **空属性**：
两个值：NULL和NOT NULL（非空）
虽然数据库默认字段基本都为空，但是实际上开发时，尽可能要保证所有的数据都不为空：**空数据没有意义，也不参与运算。**任何数字和NULL相乘都为NULL;
案例:创建一个班级表（名字，教室）
		CREATE TABLE te_class(
		name VARCHAR(20) NOT NULL,
		room VARCHAR(20) NULL  -- 代表运行为空,默认也为空
		)CHARSET utf8;
2. **注释comment**，列描述
列描述:**comment** 描述，没有实际意义，是专门来描述字段的，会根据表创建语句保存。用来给程序员（或DBA）来进行了解。 
案例：创建教师表
		CREATE TABLE te_teacher(
		name VARCHAR(20) NOT NULL COMMENT '姓名',
		money VARCHAR(20) NOT NULL COMMENT '工资'
		)CHARSET utf8;
使用`show creat table te_teacher即可查看`:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-12.jpg)
但是需要注意的是,使用desc te_teacher并不能显示:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-13.jpg)
3. **默认值default**
默认值:某一种数据会经常性的出现某一个具体的值时，可以一开始就指定好,在需要真实数据的时候，用户可以选择性的使用默认值。关键字default。
案例：创建default测试表
		CREATE TABLE te_default(
		name VARCHAR(20) NOT NULL,
		age TINYINT UNSIGNED DEFAULT 0,
		gender enum('男','女','保密') DEFAULT '男'
		)CHARSET utf8;
使用`desc te_default;`会发现已经有了默认值。
默认值的生效：数据进行插入时不给该字段赋值。
使用默认值可以不一定指定列表，故意不使用字段列表，可以使用default关键字代替：
		INSERT INTO te_default VALUES('周建新',22,DEFAULT);

---
## 6.主键
1. **增加主键**：
SQL操作中有多种方式可以给表增加主键，大致可以分为三种：
1.创建表是直接在字段后面跟primary key关键字（主键本身不允许为空）
		CREATE TABLE te_pri1(
		name VARCHAR(20) NOT NULL COMMENT '姓名',
		number CHAR(10) PRIMARY KEY COMMENT '学号: itcast + 0000 不能重复'
		)CHARSET utf8;
使用desc查看表的字段信息:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-15.jpg)
注意：PRI不一定代表主键，但是大部分时间都是代表主键。
优点：非常直接；缺点：**只能使用一个字段作为主键**。
2.在创建表的时候，在所有的字段之后，使用primary key(主键字段列表)来创建之间（如果有多个字段，则作为**复合主键**）
		CREATE TABLE te_pri2(
		number CHAR(10) COMMENT '学号：itcast + 0000',
		course CHAR(10) COMMENT '课程代码：学校代码 + 0000',
		score TINYINT UNSIGNED DEFAULT 60 COMMENT '成绩',
		-- 增加复合主键限制（学号与课程号对应并且唯一）
		PRIMARY KEY(number,course)
		)CHARSET utf8;
使用desc发现有两个主键，一个表里不可能存在两个主键，所以是复合主键：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-16.jpg)
3.表创建好之后（没有主键时），额外追加主键:可以通过修改表字段属性，也可以直接追加。
&nbsp;&nbsp;&nbsp;&nbsp;修改属性：
		alter table 表名 modify/change ...;  -- 详情见笔记一
&nbsp;&nbsp;&nbsp;&nbsp;直接追加：
		alter table 表名 add primary key(字段列表);
创建表并追加主键：
		CREATE TABLE te_pri3(
		course CHAR(10) NOT NULL COMMENT '课程编号：学校代码 + 0000',
		name VARCHAR(20) NOT NULL COMMENT '课程名字'
		)CHARSET utf8;
		-- 追加主键（方式一）
		ALTER TABLE te_pri3 modify course char(10) PRIMARY KEY COMMENT '课程编号：学校代码 + 0000';
		-- 追加主键(方式二)
		ALTER TABLE te_pri3 ADD PRIMARY KEY(course);
**追加主键的前提**:
表中数据本身是独立的（不重复，否则无法添加主键）。
2. **主键约束**
主键对应的字段中的数据不允许重复，一旦重复，则数据操作会失败（仅限增和改）。
如向te_pri1表中插入如下数据(主键冲突):
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-18.jpg)
3. **主键删除**
**没有办法更新主键：主键必须先删除才能增加。**
删除主键的方法:
		alter table 表名 drop primary key;
不管主键在哪，是什么主键，都删除。
如删除表te_pri1的主键：
		ALTER TABLE te_pri1 DROP PRIMARY KEY;
显示结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-19.jpg)
4. **主键分类**
在实际创建表的过程中，很少使用真实业务数据作为主键字段（业务主键，如学号，课程号），大部分时候是用逻辑性的字段（字段没有业务意义，值是什么都没有关系），将这种字段主键称为**逻辑主键**。
		CREATE TABLE te_pri4(
		id int PRIMARY KEY auto_increment COMMENT '逻辑主键，自增长',
		Number CHAR(10) NOT NULL COMMENT '学号,本来这个是主键',
		Name VARCHAR(20) NOT NULL COMMENT '姓名'
		)CHARSET utf8;
添加数据（未指定id）:
		INSERT INTO te_pri4(Number,Name) VALUES('1234567890','小学生');
发现id已经有值了:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-20.jpg)

---
## 7.自增长
1. **自增长简介**
自增长：当对应的字段不给值，或者说是默认值或者给NULL时，会自动被系统触发，系统会从当前字段中已有的最大值再进行+1操作，得到一个新的在不同的字段。
**自增长通常是和主键搭配的。**
2. 自增长的特点：auto_increment
&nbsp;&nbsp;&nbsp;&nbsp;a.任何一个字段要做自增长必须前提是本身是一个索引。（key一栏有值）
&nbsp;&nbsp;&nbsp;&nbsp;b.自增长字段必须是数字。（整型）
&nbsp;&nbsp;&nbsp;&nbsp;c.一张表最多只能有一个自增长。
3. 创建一个自增长表:
		-- 需要满足上述上述三个条件
		CREATE TABLE te_inc1(
		id int PRIMARY KEY auto_increment COMMENT '自动增长',
		name VARCHAR(10) NOT NULL COMMENT '姓名'
		)CHARSET utf8;
4. **自增长的使用**
		-- 触发自增长
		INSERT INTO te_inc1(name) VALUES('学生1');
		INSERT INTO te_inc1 VALUES(null,'学生2');
		INSERT INTO te_inc1 VALUES(DEFAULT,'学生3');
添加结果为:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-21.jpg)
特性:
&nbsp;&nbsp;&nbsp;&nbsp;--第一个元素默认为1，每次自增长加1。
&nbsp;&nbsp;&nbsp;&nbsp;--自增长输入值后本次失效，但是下一次开始还是能自动自增长，从最大值开始+1。
若删除学生三后在进行添加，且测试给自增长赋值：
		DELETE FROM te_inc1 WHERE id=3;
		INSERT INTO te_inc1 VALUES(null,'学生4');
		INSERT INTO te_inc1 VALUES(20,'学生5');
		INSERT INTO te_inc1 VALUES(null,'学生6');
查看结果，发现删除后还会在原来的基础上再增加:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-22.jpg)
如何确定下一次是什么增长呢？可以使用`show create table name`查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-23.jpg)
所以自增长是可以修改的。
5. 自增长的修改(两种情况):
情况1：自增长如果涉及到字段的改变，必须先删除自增长，后增加（一张表只能有一个自增长）。--修改自增长换字段
情况2：修改当前自增长已经存在的值，修改只能比当前已有的自增长的最大值大，不能小。（小不报错但是也不生效）
		-- 修改的是表选项的值
		alter table 表名 auto_increment = 0;
如修改表选项后再添加：
		ALTER TABLE te_inc1 auto_increment = 25;
		INSERT INTO te_inc1 VALUES(null,'学生7');
结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-24.jpg)
6. **自增长的变量修改**
思考：为什么自增长是从1开始且每次都只自增1呢?
所有系统的表现（如字符集，校对集等）都是由系统内部的变量进行控制的。
查看自增长对应的变量:
		show variables like 'auto_increment%';
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-25.jpg)
可以修改变量实现不同的效果:**修改是对整个数据库修改，而不是但张表。**（修改会话级:当前客户端当次连接有效）且不会改变表现有的状态。
		set auto_increment_increment=2;  -- 一次自增2(自增长步长)
		set auto_increment_offset=2;  -- 修改起始值
一般情况下不修改，没啥意义，且修改后第一次会有误差。
7. **删除自增长**:
自增长是字段的字段一个属性：可以通过modify来进行修改(保证没有auto_increment即可)：
		alter table 表名 modify 字段 类型
错误的修改:
		ALTER TABLE te_inc1 MODIFY id INT PRIMARY KEY;  -- 主键是单独存在的，见上面的show create table te_inc1
正确的修改（不要加主键名）:
		ALTER TABLE te_inc1 MODIFY id INT;
结果:自增长删除了，但是发现注释也没有了，所以得注意
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-26.jpg)

---
## 8.唯一键（unique key）
1. 介绍
一张表往往有很多字段需要唯一性，数据不能重复，但是一张表只能有一个主键，唯一键就可以解决表中有多个字段需要唯一约束的问题。
唯一键的本质与主键差不多，但是：唯一键默认自动为空，而且可以多个为空。（空字段不参与唯一性比较）
2. 增加唯一键（与增加主键差不多）
方案1：在创建表的时候，字段后面直接跟unique/unique key。
		CREATE TABLE te_uni1(
		number CHAR(10) UNIQUE COMMENT '学号，唯一，允许为空',
		name VARCHAR(20) NOT NULL
		)CHARSET utf8;
方案2：复合唯一键(主键错觉)
		CREATE TABLE te_uni2(
		number CHAR(10) NOT NULL COMMENT '学号，唯一，不允许为空',
		name VARCHAR(20) NOT NULL,
		unique key(number)
		)CHARSET utf8;
使用desc会发现number竟然有主键的标志:
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-27.jpg)
注意:number不是主键，而且该表没有主键，number又具有主键特性，所以number就被系统认为是主键了。使用show create table表名可以查看真实情况。
方案3：在创建表之后增加（追加）唯一键
		CREATE TABLE te_uni3(
		id INT PRIMARY KEY auto_increment COMMENT 'id',
		number CHAR(10) NOT NULL COMMENT '学号',
		name VARCHAR(20) NOT NULL COMMENT '姓名'
		)CHARSET utf8;
追加唯一键(和主键差不多):
		ALTER TABLE te_uni3 ADD UNIQUE KEY(number);
再使用desc te_uni3查看数据：
![](http://p5ki4lhmo.bkt.clouddn.com/00010mysql%E5%AD%A6%E4%B9%A02-28.jpg)
3. 唯一键约束：
唯一键与主键本质相同，但唯一键默认为空，且是多个为空。
如有一个表唯一键为number，可以往number中存入多个null，不会出错，而且不会冲突，若唯一键也不允许为空，那么与主键的作用一致。
4. 更新唯一键与删除唯一键：
**更新唯一键**需要先删除后增加，如果其他字段新增则不用删。
删除唯一键：
		alter table 表名 drop index 索引名字；  -- 也是删除索引的方法
唯一键默认使用字段名作为索引。如删除te_uni3表number字段上的唯一键：
		ALTER TABLE te_uni3 DROP INDEX number； 

---

