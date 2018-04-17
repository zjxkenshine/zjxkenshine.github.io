---
title: MySQL遇到的问题总结
date: 2018-04-02 20:32:11
tags: MySQL
categories: 数据库

---
## 0.问题列表
1. 游标使用时遇到的一个问题：变量与字段重名?
2. Mysql事件中如何同时指定开始时间和执行周期?
3. 查询每个月倒数第二天入职的员工?

---
## 1.问题1-5解决方法
**1)游标使用时遇到的一个问题：**
1. 出错代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-15-01.jpg)
age和height一直取不到值，还以为是没加主键，最后发现是局部变量和游标结果集重名了，后面取到的都是游标中的字段名。
2. 正确代码：
![](http://p5ki4lhmo.bkt.clouddn.com/00026mysql%E5%AD%A6%E4%B9%A08-15-02.jpg)

**2)Mysql事件中如何同时指定开始时间和执行周期?**
详细创建语法：

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
AT和EVERY同时只能有一个。

**3）查询每个月倒数第二天入职的员工?**
使用last_day()函数：表示当月的最后一天
`SELECT * FROM hire WHERE hiredate=last_day(hiredate)-1;`

---