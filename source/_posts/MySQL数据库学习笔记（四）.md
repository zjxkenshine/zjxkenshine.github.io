---
title: MySQL学习笔记（四）：高级数据操作
date: 2018-03-17 22:18:50
tags: MySQL
categories: 数据库

---
## 0.学习资料
参考书籍《深入浅出MySQL数据库开发、优化与管理维护》
参考视频《传智播客MySQL》

---
## 1.高级数据插入:
**1)主键冲突**：
1. 插入的基本语法：
		insert into 表名[(字段列表)] values(值列表);
2. 若数据插入时主键对应的值已经存在，则一定会插入失败。
当主键出现冲突的时（Duplicate Key）,可与选择性的处理：更新或者替换。
3. **更新操作：**
		Inset into 表名[(字段列表：包含主键)] values(值列表) on duplicate key update 字段 = 新值;
注意update后不需要set。
创建一个`te_student`表：
		CREATE TABLE IF NOT EXISTS te_stu1(
		id INT auto_increment PRIMARY KEY COMMENT '主键',
		code CHAR(10) NOT NULL  COMMENT '学号',
		name VARCHAR(20) NOT NULL COMMENT '姓名'
		)ENGINE=INNODB DEFAULT CHARSET utf8;
插入三条数据：
		INSERT INTO te_stu1 VALUES(1,"1111111111","学生1");
		INSERT INTO te_stu1 VALUES(2,"1111111112","学生2");
		INSERT INTO te_stu1 VALUES(3,"1111111113","学生3");
如若要更新第二条数据，一般情况下会失败，需要这样写：
		INSERT INTO te_stu1 VALUES(2,"2222222222","学生22")
		ON DUPLICATE KEY UPDATE code='2222222222';
会显示有两行受影响，插入一行，更新一行。但是只会更新一个字段。
但是如果有几个字段需要修改，用and连接我测试是无效的，可以使用替换，或者，还是直接更新吧。
4. **替换：**
先删除后插入：
		replace into 表名[(字段列表：包含主键)] values(值列表);
若主键冲突，则替换，不冲突则直接插入。
如执行：
		REPLACE INTO te_stu1 VALUES(2,"2222222222","学生22");
		REPLACE INTO te_stu1 VALUES(4,"4444444444","学生4");
结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-01.jpg)

**2)蠕虫复制：**
从已有的数据中去获取数据，然后将数据进行新增操作，数据成倍的增加。
1. 表创建的高级操作：从已有的表创建新表（复制表结构）
		Create table 表名 like 数据库.表名；   -- 同一个数据库不用加数据库名
如复制一个te_stu1：
		CREATE TABLE te_stu1_copy LIKE te_stu1;
只会复制表结构，但是没有数据。
2. 蠕虫复制：
先查出数据，然后将查出的数据新增一遍：
		Insert into 表名[(字段列表)] select 字段列表[*] from 数据表名；
如将te_stu1的数据搬到te_stu1_copy:
		INSERT INTO te_stu1_copy SELECT * from te_stu1;
te_stu1_copy中已经有数据了：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-02.jpg)
如果没有复制主键的话，一直执行蠕虫复制就会是数据成倍增加(复制自身):
		INSERT INTO te_stu1_copy(code,name) SELECT code,name from te_stu1_copy;
3. 蠕虫复制的意义：
	- 从已有表中拷贝数据到新表中
	- 可以迅速的让表中的数据膨胀，膨胀到一定的数量级来测试表的压力

---
## 2.高级更新与删除数据
**1)更新数据：**
1. 基本语法：
		Update 表名 set 字段 = 值 [where 条件]；
2. 高级新增（限制记录条数）
		Update 表名 set 字段 = 值 [where 条件][limit 更新数量]；
使用上述蠕虫复制多复制几条记录，如要更新前5个学生的学号为0000000000:
		UPDATE te_stu1_copy SET code = '0000000000' LIMIT 5;
结果如下:
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-03.jpg )

**2)删除数据：**
删除数据的限制与更新类似:（通过limit限制数据）
1. 语法：
		delete from 表名 [where 条件][limit 数量]
2. 案例，删除te_stu1_copy的前3个学生1
		DELETE FROM te_stu1_copy WHERE name='学生1' LIMIT 3;
结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-04.jpg)

**3)删除并还原自增长：**
1. 完全删除数据后自增长`auto_increment`表选项仍为之前的自增长，可以使用：
		ALTER TABLE 表名 auto_increment=1;
来修改，但是显得太麻烦了。可以直接删除表后再创建。
2. 表重建：
		Tuncate 表名；
先删除该表，再创建该表（先清空表再重置自增长）如：
		TRUNCATE te_stu1_copy;
运行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-05.jpg)

---
## 3.数据查询（select表选项，字段别名，数据源）
简单查询：
`Select 字段列表/* from 表名 [where 条件];`
完整查询：

	select[select选项] 字段列表[字段别名]/* from 数据源 [where 条件子句] [group by子句][having 子句][order by子句][limit 子句]
只要有这些子句顺序必须得是这样。
**1)select选项：**
1. select对查出来的结果的处理方式，有俩选项：
All：默认的，保留所有信息
Distict：去重（所有字段都相同才算重复）
2. 案例：
新建表te_stu2复制表te_stu1的结构但是删除主键与自增长，并使用蠕虫复制多条数据：
		CREATE TABLE te_stu2 LIKE te_stu1;
		ALTER TABLE te_stu2 DROP PRIMARY KEY;
		ALTER TABLE te_stu2 modify id INT;
		INSERT INTO te_stu2 SELECT * from te_stu1;  
		INSERT INTO te_stu2 SELECT * from te_stu2;   //使用多次
查看一下共有多少条数据以及查询：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-06.jpg)
注意：不能针对某一字段去重。

**2)字段别名：**
1. 为什么使用别名：
当数据进行查询操作的时候，有时候名字并不一定满足需求（多表查询时会有重名字段）:需要对字段名进行重命名：别名。
2. 重命名的方式：
		字段名 [as] 别名          -- as可以省略
3. 测试：
先往te_stu1表中插入学生5和学生6，查询学生4并用`number`代替`code`字段：
		SELECT id,code number,name FROM te_stu1 WHERE name='学生4';
结果中的code变为了number:
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-07.jpg)

**3)数据源：**
1. 数据的来源，关系型数据库的数据来源都是表，只要保证数据类似二维表的都可以。
数据源分类：单表数据源，多表数据源，查询语句
2. 单表数据源：
		select * from 表名
3. 多表数据源：
		select * from 表名1，表名2，表名3...
如表1有3条记录，表2有4条记录，则不加条件的`select * from 表名1,表名2;`会查询出3\*4=12条数据。
从一张表中取出一条数据去另外一张表中匹配所有记录，而且全部保留，这种结果称为：交叉连接（笛卡尔积）。
**应该尽量少用笛卡尔积。**
4. 查询语句：
子查询：数据的来源是一条查询语句。
		select * from (select 子查询) as 表名；
如查询te_stu2去重后的结果中的学生22：
		SELECT * FROM (SELECT DISTINCT * FROM te_stu2) AS s WHERE s.name='学生22';
运行结果为：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-08.jpg)
`as`同样可以给数据源提供别名。

---
## 4.数据查询(where,groupby子句）
**1)Where子句：**
1. where子句:
用来判断数据，删选数据
返回结合：0或1，代表false和true
2. where 的判断条件：
比较运算符：>,<,>=,<=,!=,<>,<=>,like,between and,in/not in;
逻辑运算符：&&(and),||(or),!(not)
详细的关于运算符的介绍可以查看[笔记3](https://zjxkenshine.github.io/2018/03/16/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E4%B8%89%EF%BC%89/) 
3. where原理：
where是唯一一个直接从磁盘获取数据的时候就开始判断的条件：
从磁盘取出一条数据，进行where判断，成立则保存到内存，失败则直接放弃。
4. 案例：
给te_stu1添加age字段与身高height字段：
		ALTER TABLE te_stu1 ADD age TINYINT UNSIGNED;
		ALTER TABLE te_stu1 ADD Height TINYINT UNSIGNED;
给每个学生添加信息（随机）:
		UPDATE te_stu1 SET age=FLOOR(RAND()*10+20),height=FLOOR(RAND()*30+160);
查看添加结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-09.jpg)
找出学生id为1，3，5的学生：
		SELECT * FROM te_stu1 WHERE id=1||id=3||id=5;  -- 逻辑运算
		SELECT * FROM te_stu1 WHERE id IN(1,3,5);  -- 落在集合
查询身高在165-180之间的学生：
		SELECT * FROM te_stu1 WHERE height>=165 && height<180;
		SELECT * FROM te_stu1 WHERE height BETWEEN 165 AND 180;
注意:between本身是闭区间，且左边的值必须小于等于右边的值。
查询所有(有时为保证SQL语句的完整性)：
		SELECT * FROM te_stu1 WHERE 1;

**2)group by子句:**
1. 分组：根据某个字段进行分组，相同的放一组，不同的放一组。
2. 基本语法：
		group by 字段名
3. 测试：
给te_tu1添加一个sex字段:
		ALTER TABLE te_stu1 ADD sex enum('男','女');
给所有学生添加性别（随机）:
		UPDATE te_stu1 SET sex=FLOOR(RAND()*2+1);
根据性别分组：
		SELECT * FROM te_stu1 GROUP BY sex;
结果是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-10.jpg)
只有两条数据?没错只有两条，按分组字段是为了数据统计。统计函数如下：
4. 统计函数:
	>Count()：统计分组后的记录数
	>Max()：统计每组中的最大值
	>Min()：统计最小值
	>Avg()：统计平均值
	>Sum()：统计和
5. 统计函数测试：
		SELECT COUNT(*),MAX(age),MIN(height),AVG(height),Sum(age),sex FROM te_stu1 GROUP BY sex;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-11.jpg)
一旦用到了`group by`子句最好就使用统计，否则groupby将没有意义。
6. 关于统计函数：
count()里面可以使用两种参数：\*和字段(**null不参与运算**)。
avg()中的null字段也算入总数。
7. 分组会自动排序，且默认是升序：
		group by 字段 [asc|desc];  -- 对分组之后的结果进行排序（asc升,desc降）

**3)多字段分组:**
1. 多字段分组：
先根据一个字段进行分组，然后对分组后的结果再次按照其他字段进行分组。
2. 测试：
先往te_stu1下再添加四个学生，代码略。
再添加一个class字段:
		ALTER TABLE te_stu1 ADD class enum('班级1','班级2');
添加每个人的班级(随机)：
		UPDATE te_stu1 SET class=FLOOR(RAND()*2+1);
先按班级分组再按性别分组，查找每个班对应性别有多少人：
		SELECT class,sex,COUNT(*) FROM te_stu1 GROUP BY class,sex;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-12.jpg)
3. group_concat()方法:
可以对分组的结果中的某个字段进行进行字符串连接（会输出该字段的连接值），用法为：
group_concat(字段)，如：
		SELECT class,sex,COUNT(*),group_concat(name) FROM te_stu1 GROUP BY class,sex;
运行结果为:
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-13.jpg)

**4）回溯统计with rollup**
1. 原理:
任何一个分组后都会有一个小组，最后都会向上级进行汇报统计(根据当前分组的字段)
这就是回溯统计，回溯统计时会将分组字段置空。
2. 测试：
		SELECT class,COUNT(*) FROM te_stu1 GROUP BY class;   -- 一个字段，分为两组，共10人
		SELECT class,COUNT(*) FROM te_stu1 GROUP BY class WITH ROLLUP; -- 回溯统计，会加一个上级的总人数
		SELECT class,sex,COUNT(*),group_concat(name) FROM te_stu1 GROUP BY class,sex WITH ROLLUP;  -- 层层回溯，会加上各班级总人数（第二级）和总人数(第一级)
一层分组的回溯统计：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-14.jpg)
两层分组的回溯统计：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-15.jpg)

---
## 5.数据查询(having,orderby,limit子句）
**1)Having子句:**
1. 说明：
与where子句，也是进行条件判断的。
where是针对磁盘数据进行判断，进入到内存之后，会进行分组操作，分组的操作就需要having来处理。
having能做where能做的几乎所有事情，但是很多having能做的事where做不了：
	- **分组统计的结果或者统计函数的值都只有having能够操作。**
	- having能使用字段别名而where不行
2. 基本语法：
		select ... having 条件
3. 案例：
再次添加5个学生，并且添加班级3，并修改班级更新语句：
		UPALTER TABLE te_stu1 MODIFY class enum('班级1','班级2','班级3');
		UPDATE te_stu1 SET class=FLOOR(RAND()*3+1);
求出所有班级人数大于5的班级（使用别名sum）：
		SELECT class,COUNT(*) AS sum FROM te_stu1 GROUP BY class HAVING sum >5;
运行结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-16.jpg)
这样是非法的:
		SELECT class,COUNT(*) AS sum FROM te_stu1 WHERE sum >5 GROUP BY class;  -- 非法，where不能使用count(*)及别名
where加载时数据还没到内存，count(*)与别名sum都还没加载。

**2）order by子句：**
1. 说明：
排序，根据某个字段进行升序(asc)或者降序(desc)排序，依赖校对集
2. 基本用法：
		select ... Order by 字段名 [asc|desc]  -- 默认asc升序
3. 与group by的区别：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-17.jpg)
分组是为了排序，而group by是为了统计。
4. 多字段排序：
排序可以进行多字段排序，先根据某一个字段排好序，然后排序好的内部再按某个数据次序进行排序。
如先按班级排序(升序）再按照年龄排序(降序):
		SELECT * FROM te_stu1 ORDER BY class [ASC],age DESC;
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00014mysql%E5%AD%A6%E4%B9%A04-18.jpg)

**3)limit子句:**
1. 说明：
是一种限制结果的语句：本质是限制数量。
2. 有两种使用方式：
	- 只用来限制长度（数量）:`limit 数据量`
	- 限制起始位置，限制数量：`limit 起始位置(offset)，长度(length)`(从指定的位置向后找几个数据)
3. 案例测试:
查询学生前两个：
		SELECT * FROM te_stu1 LIMIT 2;
		SELECT * FROM te_stu1 LIMIT 0,2;  -- 编号是从0开始的
从第三条开始查两条：
		SELECT * FROM te_stu1 LIMIT 2,2;
4. 主要作用：**实现数据的分页**
分页：为用户节省时间，提高服务器响应效率，减少资源浪费。

5. 分页的简单理论实现
	- Page model类需要的属性：
	totalCount：总的记录数量
	totalPage：总的记录数量
	//前俩用于前台首末页等功能，后俩用于分页
	PageSize：每页显示数量（最好固定）
	NowPage:当前所在哪页
	- Length=每页显示的数据量：基本不变
 	Offset=(页码数量-1)*每页显示数量
	- 简单的查询语句这样写：
	`select * from ***  limit "+(page.getNowPage()-1)*page.getPageSize()+","+page.getPageSize();`

---