---
title: MySQL学习笔记（五）：连接、联合查询，子查询及外键
date: 2018-03-21 09:55:06
tags: MySQL
categories: 数据库

---
## 0.学习资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

---
## 1.连接查询（交叉连接与内连接）
**1)连接查询准备（了解）**
1. 创建学生与班级表,且是多对一的关系，并且添加信息：
		-- 连接查询准备
		-- 创建学生表
		CREATE TABLE te_stu3 LIKE te_stu1;
		ALTER TABLE te_stu3 CHANGE class classid TINYINT UNSIGNED;
		
		-- 插入数据
		INSERT INTO te_stu3(CODE,NAME) VALUES('0000000001','学生1'),('0000000002','学生2'),('0000000003','学生3'),('0000000004','学生4'),
		('0000000005','学生5'),('0000000006','学生6'),('0000000007','学生7'),('0000000008','学生8'),('0000000009','学生9'),('0000000010','学生10');
		-- 随机身高，性别，年龄，班级id(1,2,3)
		UPDATE te_stu3 SET sex=FLOOR(RAND()*2+1),height=FLOOR(RAND()*30+160),age=FLOOR(RAND()*10+20),classid=FLOOR(RAND()*3+1);
		-- 创建班级表
		CREATE TABLE te_class1(
		id TINYINT PRIMARY KEY auto_increment COMMENT '班级id',
		cla_name VARCHAR(10) NOT NULL COMMENT '班级'
		)ENGINE=INNODB DEFAULT CHARSET utf8;
		-- 添加班级信息
		INSERT INTO te_class1 VALUES(null,'班级一'),(null,'班级二'),(null,'班级三');
2. 连接查询：
将多张表(可以大于两张)进行记录的连接(按照某个特定的条件进行数据拼接)。
最终的结果是：记录数可能有变化，字段数一定会增加(至少两张表)
关键字：join,使用方式:
		左表 join 右表
3. 连接查询的意义：
在用户查看数据的时候，需要显示的数据来自多张表。
4. 连接查询分类：
内连接
外连接
自然连接
交叉连接

**2）交叉连接：**
1. 交叉连接：cross join从一张表中循环取出每一条记录，每条记录都去另一张表中进行匹配，匹配一定保留（没有条件的匹配）,连接本身字段会增加，最后形成的结果： 笛卡尔积
2. 基本语法：
		feom 左表 cross join 右表
等价于：
		from 左表，右表
注意，连接后的表作为整个数据源。
3. 案例：
		-- 交叉连接
		SELECT * from te_stu3 CROSS JOIN te_class1;
		SELECT * from te_stu3,te_class1;
查询结果(条数)：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-01.jpg)
4. 注意：
笛卡尔积没有意义，应该尽量避免。
只是为了保证结构的完整性。一般不用。

**3)内连接：**
1. 内连接：[innner] join,从左表中取出每一条记录去右表中匹配条件，且必须是左表与右表某个条件相同最终才会保留结果，否则不保留。
2. 基本语法:
		左表 [inner] join 右表 on 左表.字段 = 右表.字段
on表示连接条件:条件字段就是代表相同的业务含义(如te_stu3表中的classid与te_class1表中的id)
3. 案例:
		-- 内连接
		SELECT * FROM te_stu3 stu JOIN te_class1 cla ON stu.classid=cla.id;
将`stu.classid=cla.id`改为`classid=cla.id`也可，前提是字段要有唯一性。
结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-02.jpg)
发现有两个id,第二个因为使用Navicate运行自动加上了1，使用cmd运行时不会加。
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-03.jpg)
所以最好是建字段时加上表名前缀加以区分，或者使用字段别名。
4. 别名使用:
在查询数据时不同的表有同名字段时，这个时候需要加上表名才能区分，而表名太长，需要别名，以及字段别名才能区分。
当然像这种id和classid值相同的情况，只留下一列就可以了（假设name重名）：
		SELECT s.*,c.cla_name as c_name FROM
		te_stu3 s JOIN te_class1 c ON s.classid=c.id;
运行结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-04.jpg)
5. 注意：
内连接只会保留满足条件的字段，如果没有条件（on）,则会变成交叉连接。
内连接还可以使用where代替on关键字:
		select * fron 表1 join 表2 where 条件；
		select * fron 表1，表2 where 条件；
但是通常不用where：
**where效率没有on高**

---
## 2.连接查询(外连接，自然连接)：
1. 外连接（outer join）： join
指以某张表为主取出里面的所有记录，每条与另外一张表表进行连接，不管能不能匹配上条件，最终都会保留：能匹配正确保留；**不能匹配,其他表的字段都置空。**
2. 外连接分为两种：以某张表为主
left join..on：左连接，以左表为主；
right join..on：右连接，以右表为主。
3. 基本语法:
		左表 [left|right] join 右表 on 左表.字段=右表.字段
**外连接的on必须要有**，必须要有条件。
4. 示例：
往班级表添加班级四并且修改某两个学生班级ID为0：
		-- 外连接准备
		INSERT INTO te_class1 VALUES(NULL,'班级4');
		UPDATE te_stu3 SET classid=0 WHERE id in (5,6);
执行左连接:
		SELECT s.*,c.cla_name as c_name FROM
		te_stu3 s LEFT JOIN te_class1 c on s.classid=c.id;
运行结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-05.jpg)
记录数>=左表。两个学生无法匹配到班级，所以用null代替匹配的值。
执行右连接:
		SELECT s.*,c.cla_name as c_name FROM
		te_stu3 s RIGHT JOIN te_class1 c on s.classid=c.id;
运行结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-06.jpg)
记录数>=右表，班级四没有人匹配，所以其他字段置空。
5. 注意：
左连接和右连接有主表差异，但是显示的结果永远是左表数据在左，右表数据在右。

**2)自然连接：**
1. 自然连接：natural join,自动匹配连接条件。
系统以字段名字作为匹配模式，同名字段作为匹配条件，多个字段同名则都作为匹配条件。
2. 分类：
自然内连接，自然外连接。
其实和内连接外连接一样，就是条件自动找。
		左表 natural [left|right] join 右表
3. 自然内连接：
准备(将te_class1的id字段改为classid)：
		ALTER TABLE te_class1 CHANGE id classid TINYINT UNSIGNED auto_increment;  -- 自增长需重新指定，主键不用.
		SHOW CREATE TABLE te_class1;
使用自然内连接：
		SELECT * FROM te_stu3 NATURAL JOIN te_class1;
结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-07.jpg)
重名的字段连接后自动合并。
4. 自然外连接：
		SELECT * FROM te_stu3 NATURAL LEFT JOIN te_class1;
		SELECT * FROM te_stu3 NATURAL RIGHT JOIN te_class1;
5. 需要保证表的严格规范，一般不用自然连接。
内连接和外连接都可以模拟自然连接。
		左表  [left|right] join 右表 USING(字段名)
使用左连接模拟自然连接：
		SELECT * FROM te_stu3 LEFT JOIN te_class1 USING(classid);
运行结果（之前已将学生5班级id改为null）:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-08.jpg)

---
## 3.外键(增删与默认约束)
**1)外键简介与新增：**
1. 外键(foreign key)：
foreign key：外键，外面的键（键不在自己表中）：
如果一张表中有一个字段(非主键)指向另一张表的主键，则该字段称为外键。
外键必须满足一些要求才可以称为外键(后面会说)。
**一张表可以有多个外键**
2. 增加一个外键：
外键可以在创建表的时候新增或者创建表之后增加。但是要考虑数据的问题（约束）。
3. 创建表的时候增加外键：
		create table 表名（
		字段 类型 列属性,
		...
		字段 类型 列属性,
		[constraint 外键名字] foreign key(外键字段) references 外部表（主键字段）
		）charset utf8;
先创建te_class2表：
		CREATE TABLE te_class2(
		cla_id TINYINT UNSIGNED auto_increment PRIMARY KEY,
		cla_name VARCHAR(10) NOT NULL COMMENT '班级名'
		)CHARSET utf8;
创建te_stu4表并指定外键为cla_id：
		CREATE TABLE te_stu4(
		stu_id INT UNSIGNED auto_increment PRIMARY KEY,
		stu_name VARCHAR(10) NOT NULL COMMENT '姓名',
		cla_id TINYINT UNSIGNED,
		FOREIGN KEY(cla_id) REFERENCES te_class2(cla_id)
		)CHARSET utf8;
此时使用desc查看te_stu4表:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-09.jpg)
发现外键处多了MUL。使用`show create table te_stu4`查看建表语句:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-10.jpg)
发现从先key了cla_id然后再使用了外键。
**外键：要求字段本身必须是索引，如果不是索引的话，就会先创建索引（key）,然后再创建外键。**
重点看倒数第二句:
		  CONSTRAINT `te_stu4_ibfk_1` FOREIGN KEY (`cla_id`) REFERENCES `te_class2` (`cla_id`)
`CONSTRAINT `指定的是外键名字（可修改）,`FOREIGN KEY `后面跟的是外键字段，`REFERENCES `后面是外键引用（其他表的主键）
4. 在新增表之后增加外键：
		Alter table 表名 add [constraint 外键名字] foreign key(外键字段) references 父亲（主键字段）
创建没有外键的学生表te_stu5：CREATE TABLE te_stu5(
		stu_id INT UNSIGNED auto_increment PRIMARY KEY,
		stu_name VARCHAR(10) NOT NULL COMMENT '姓名',
		cla_id TINYINT UNSIGNED
		)CHARSET utf8;
为te_stu5增加外键：
		-- 增加外键
		ALTER TABLE te_stu5 ADD 
		-- 指定外键名
		CONSTRAINT stu_class_1
		-- 指定外键字段
		FOREIGN KEY(cla_id)
		-- 引用父表主键
		REFERENCES te_class2(cla_id);
使用desc查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-11.jpg)

**2)删除外键：**
1. 外键不能修改，只能先删除，后新增。
2. 删除外键语法：
		Alter table 表名 drop foreign key 外键名；
		 -- 一张表中可以有多个外键但是名字不能相同
3. 删除te_stu5的外键:
		ALTER TABLE te_stu5 DROP FOREIGN KEY stu_class_1;
提示删除成功，但是查看表结构会发现：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-11.jpg)
根本没有变化，但是查看建表语句，会发现已经删掉了：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-12.jpg)
之所以表结构看不到是因为外键创建之前变为了索引（key）。
所以**外键的删除不能在表结构中看到，要看建表语句。**

**3)外键默认作用：**
1. 外键的默认作用有两种：
&nbsp;&nbsp;&nbsp;&nbsp;a.对子表的约束（外键字段所在的表）：子表进行写操作，如果对于的外键字段在父表中找不到对应的匹配，那么操作会失败。（约束子表的数据操作）
&nbsp;&nbsp;&nbsp;&nbsp;b.对父表的约束：父表数据进行操作（删和改：都涉及到主键本身时），如果对应的主键在子表中被数据所引用，则删和改都会失败。
2. 对子表的约束测试：
先给te_class2添加数据：
		INSERT INTO te_class2 SELECT * FROM te_class1;
然后给te_stu4添加一个班级id为5（不存在）的学生：
		INSERT INTO te_stu4 VALUES (null,'学生一',5);
会报错：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-13.jpg)
不能更新子表的行，原因是父表没有id=5的班级。
3. 对父表的约束测试：
给te_stu4添加对的数据：
		INSERT INTO te_stu4 VALUES (null,'学生一',2),(null,'学生二',3);
删除：
		DELETE FROM te_class2 WHERE cla_id=2; -- 失败
结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-14.jpg)
更新：
		UPDATE te_class2 SET cla_id=5 WHERE cla_id=2;  -- 失败，已经被学生引用
		UPDATE te_class2 SET cla_id=5 WHERE cla_id=1;  -- 成功，未被学生引用
这里就不运行代码了。

---
## 4.外键（外键条件与约束）
**1)外键的条件：**
1. 外键存在的条件:
	- **保证表的存储引擎是innoDB**,非innoDB存储引擎,外键可以创建成功但是没有约束效果。
	- **外键字段的列类型必须与父表的主键字段类型一致**。
	- 一张表中外键名字不能重复。
	- 增加外键的字段(数据已经存在)，必须保证数据与父表主键对应。

2. 如在te_stu5(外键已删)中插入数据：
若添加如下(cla_id范围为1234)：
		INSERT INTO te_stu5 VALUES (null,'学生一',2),(null,'学生二',3);
则可以在cla_id字段新增外键。
若添加了如下数据：
		INSERT INTO te_stu5 VALUES (null,'学生三',5);
则无法新增外键。会报错。
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-15.jpg)
数据已经存在，新增外键时无法改变不能匹配的事实，所以添加失败。

**2)外键约束：**
1. 所谓的外键约束：就是指外键作用
之前的约束是外键的默认作用，可以对外键的需求，进行定制操作--**外键约束模式**。
但是，对子表的约束无法更改。
2. 有三种约束模式（都是对父表的约束，操作都指有关主键的操作）：
	- District：严格模式：父表不能更新或删除一个已经被字表数据应用的记录。
	- Caseade：级联模式：父表的操作，对应子表关联的数据也跟着被删除、修改。
	- Set null：置空模式：父表操作之后，子表对应的数据（外键字段）被置空。
3. 默认情况下更新与删除都是严格模式。
通常一个合理的做法（约束模式）：
**删除的时候子表置空，更新的时候子表级联操作。**
4. 语法：
		Foreign key(外键字段) references 父表（主键字段） on delete 删除模式 on update 更新模式；

**3)外键约束测试：**
1. 创建表te_stu6并指定模式：删除置空，更新级联
		-- 外键约束
		CREATE TABLE te_stu6(
		stu_id INT UNSIGNED PRIMARY KEY auto_increment,
		stu_name VARCHAR(10) NOT NULL COMMENT '姓名',
		cla_id TINYINT UNSIGNED,
		CONSTRAINT stu6_cla2 FOREIGN KEY(cla_id) REFERENCES te_class2(cla_id)
		-- 指定删除模式
		ON DELETE SET NULL
		-- 指定更新模式
		ON UPDATE CASCADE
		)CHARSET utf8;
查看表创建语句`show create table te_stu6`：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-16.jpg)
模式已经指定好了。
2. 上述模式测试：
添加学生信息：
		INSERT INTO te_stu6 VALUES (NULL,'学生1',1),(NULL,'学生2',2),(NULL,'学生3',3),(NULL,'学生4',3),(NULL,'学生5',4),(NULL,'学生6',1);
更新父表主键（将4班变为5班）：
		UPDATE te_class2 SET cla_id=5 AND cla_name="班级5" WHERE cla_id=4;
发现可以修改，查询学生表的数据,发现学生5级联修改：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-17.jpg)
然后删除班级5（删除父表主键）：
		DELETE FROM te_class2 WHERE cla_id=5;
再此查看学生表te_stu6的数据：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-18.jpg)
发现学生5的外键置为NULL了。
3. 删除置空的前提条件：
外键字段允许为空。若不满足条件则外键无法创建。
4. 需要注意的地方：
如果一个表的主键同时被多个表外键引用，那么这张表的更新删除时只要有一个外键不满足，就操作不成功。
外键虽然很强大，能进行各种约束，但是降低了数据的可控性。
5. 可控性：
在实际开发中很少使用外键来处理。但是外键很重要。

---
## 5.联合查询:
**1)语法及介绍：**
1. 联合查询：
将多次查询（多条select语句）,在记录上进行拼凑（字段不会增加）
由多条select语句构成，每一条select语句获取的字段数必须严格一致(字段类型无关)。
2. 基本语法：
		select 语句1 Union[union 选项] select 语句2 [Union...]
可以多个表联合查询。
3. union选项，有俩：
All：保留所有（不管重复）；
Distinct：去重（整个重复），默认值。
4. 简单的测试：
去重与不去重：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-19.jpg)
字段数量一致就可以，其他都不用管:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-20.jpg)

**2）联合查询的意义：**
1. 查询同一张表，但是需求不同。
如查询同一张学生表，要求男生身高升序，女生身高降序。
2. 多表查询（开发中必然会涉及）：
多张表的结构是完全一样的，保存的数据的结构也是完全一样的。
数据太多（如腾讯的QQ号）必须分表，而且分表后表的结构完全相同。

**3)男生身高升序，女生身高降序的实现：(orderby正确用法)**
1. 理论上的实现：
		SELECT * FROM te_stu1 WHERE sex='男' ORDER BY Height ASC
		UNION
		SELECT * FROM te_stu1 WHERE sex='女' ORDER BY Height DESC;
但是会报错且告诉你用法错误:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-21.jpg)
2. **order by**的使用：
a.**在联合查询中order by不能直接使用，需要使用（）**，但仅这一步未生效。
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-22.jpg)
b.**若要order by生效必须搭配limit：**使用limit限定的自定义条数。
正确的用法：
		(SELECT * FROM te_stu1 WHERE sex='男' ORDER BY Height ASC LIMIT 9999999999)
		UNION
		(SELECT * FROM te_stu1 WHERE sex='女' ORDER BY Height DESC LIMIT 9999999999);
查看结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-23.jpg)
这是硬性搭配，牢记。

---
## 6.子查询
### 子查询分类及标量子查询
1)什么是子查询:
子查询：sub query
查询是在某个结果之上进行的：(一条select语句内部包含另外一条select语句)
**2)子查询的分类：**
有两种分类：按位置分，按结果分
1. 按位置分：子查询（select语句）在外部查询中出现的位置。
	- from子查询：子查询跟在from之后
	- wher子查询：子查询出现在Where条件中
	- Exists子查询：子查询出现在exists中
2. 按结果分类（理论上讲任何一个查询结果都是二维表）：
	- 标量子查询：子查询得到的结果是一行一列
	- 列子查询：子查询得到的结果是一列多行
	- 行子查询：子查询得到的结果是一行多列（多行多列）
	- 表子查询：子查询得到的结果是多行多列的（出现的位置在from之后）

**3)标量子查询：**
1. 需求字段：
获取te_class1中班级三的所有学生（te_stu3）。
2. 步骤：
确定数据源（所有学生）：
		select * from te_stu3 where classid=?;
获取班级id:
		select classid from te_class1 where cla_name='班级三';
查询语句：
		SELECT * FROM te_stu3 WHERE classid=(SELECT classid from te_class1 WHERE cla_name='班级三');
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-24.jpg)
3. 查询出的id是一行一列的。

### 列子查询与行子查询
**1）列子查询：**
1. 需求：
查询所有在读班级的学生（班级表中存在的班级）。
2. 步骤：
确定数据源：学生
		select * from te_stu3 where classid in(?);
获取班级id:
		select classid from te_class1;
查询语句：
		select * from te_stu3 where classid in(select classid from te_class1);
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-25.jpg)
3. 列子查询返回的结果会比较长，需要使用in作为条件匹配，在mysql中还有几个类似的条件：**all，some，any**。
any，等于其中一个即可，=Any等价于in，any和some是一样的。
all，等于其中的全部。
如:
		select * from te_stu3 where classid =ANY(select classid from te_class1);
		select * from te_stu3 where classid =SOME(select classid from te_class1);
效果和in相同。
		select * from te_stu3 where classid =all(select classid from te_class1);
显示无结果。
否定时：
		select * from te_stu3 where classid !=ANY(select classid from te_class1);
		select * from te_stu3 where classid !=SOME(select classid from te_class1);
显示的是所有结果（null除外）。
		select * from te_stu3 where classid！=all(select classid from te_class1);
显示和in刚好相反。本表来说只有班级id为0和null。

**2)行子查询：**
1. 可以使一行多列也可以是多行多列（构造行元素）
2. 需求：
查询出学生中年龄最大或身高最高的学生。
注意：where后面不能用max等函数，因为数据未到内存。
3. 步骤：
确定数据源：
		select * from te_stu3 where age=? OR height=?;
确定最大年龄与身高：
		select max(age),max(height) from te_stu3;
可以实现的一个语句：
		select * from te_stu3 where
		age =(select max(age) from te_stu3)
		AND
		Height=(select max(height) from te_stu3);
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-26.jpg)
但是这样太麻烦了，可以构造行元素。
4. **构造行元素**：
		select * from te_stu3 where
		(age,Height) =(select max(age),max(height) from te_stu3);
(age,Height)代表的就是一个行元素。
运行结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-27.jpg)
5. 另一种方法使用order by和limit，但是效率低，且有很多问题。

### 表子查询与exsists子查询
**1)表子查询：**
1. 概念：
返回的结果是当做一个二维表来使用的。
2. 要求：
找出每个班中最高的一个学生
3. 步骤：
确定数据源（暂时无法确定）
先将学生按身高降序排序：
		SELECT * FROM te_stu3 ORDER BY Height DESC;
每个班选出第一个学生(group by只保留每类第一个)：
		SELECT * FROM te_stu3 GROUP BY classid;
表子查询：将得到的结果作为from数据源（from子查询）
		SELECT * FROM （SELECT * FROM te_stu3 ORDER BY Height DESC） as stu GROUP BY classid;  -- 要加别名构建表
结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-28.jpg)

**2)exists子查询：**
1. 介绍：
exists:是否存在的意思，exists子查询就是用来判断某些条件是否满足（跨表）
exists是接在where之后的。exists返回的结果为0和1。
2. 关于exists的测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-29.jpg)
3. exist子句的需求：
查询所有班级存在的学生。
4. 步骤：
确定数据源：
		Select * from te_stu3 where ?;
确定条件是否成立(有数据为1，无数据为0)
		Exists(Select * from te_class1);
exists子查询:
		Select * from te_stu3 where Exists(Select * from te_class1);
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00016mysql%E5%AD%A6%E4%B9%A05-30.jpg)

---

