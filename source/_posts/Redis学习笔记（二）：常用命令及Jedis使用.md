---
title: Redis学习笔记（二）：key命令，Jedis使用以及String字符串类型操作
date: 2018-4-10 14:59:03 
tags: Redis
categories: NoSQL

---
## 0.学习资料
参考视频：《动力节点Redis视频教程》
参考网址：<http://www.redis.net.cn/tutorial/3501.html>
Redis命令参考手册：
<http://redisdoc.com/>

---
## 1.Redis的简单使用
**1)准备工作：**
1. 首先需要开启redis服务：指定配置文件，后台启动
		# nohup redis-server redis.config &
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-03.jpg)
2. 登录到命令行客户端：
		# redis-cli
如果配置了密码需要先使用auth输入密码才能进入客户端操作。
3. 关于redis的库：
**Redis默认有16个库**,默认使用的是0号库。
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-02.jpg)
这个在redis的配置文件redis.conf中可以找到：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-04.jpg)
可以自行修改，但是最好不要修改。

**2)常用的Redis命令：(需要在cli等客户端中使用)**
所有的Redis命令有自动补齐功能，按TAB键就可以了。
1. `ping`： 测试服务是否可用，如果返回pong则说明服务器正常
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-01-1.jpg)
2. `select`：切换库，如果要切换到5号库就是用`select 5`
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-05.jpg)
3. 删除数据:
删除所有库的数据：`flashall`
删除当前数据库内的数据：`flashdb`
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-06.jpg)
4. `config get *`：获得redis的所有配置值
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-07.jpg)
我的配置有178行，其余的就不截图了。
这些配置项很重要，后面也会多次设置。
5. 退出客户端cli:
`exit`或者`quit`
6. 查看当前数据库中有多少个key：`dbsize`
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-08.jpg)
7. 查看Redis服务器的统计信息：`info`
在后面主从集群时会涉及。
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-09.jpg)

---
## 2.Redis的key命令
**1)关于redis的命令简介及命令手册：**
1. redis的常用数据操作命令为1+5：
1+5：一种key的操作，5种不同数据类型的操作
5种数据类型：
String（字符串），Hash(哈希表)，List(列表)，Set(集合)，SortedSet（有序集合）
2. 这里主要介绍的是key的操作，5种数据类型的命令操作后面会分别介绍。
3. 命令手册：
英文版：
<https://redis.io/commands>
中文版：
<http://redisdoc.com/>

**2)Redis的常用Key命令**
1. keys命令：后面加匹配模型
`keys *`：列出该数据库中的所有键
`keys h?llo`： 匹配 hello ， hallo 和 hxllo 等。
`keys h*llo`： 匹配 hllo 和 heeeeello 等。
`keys h[ae]llo`： 匹配 hello 和 hallo ，但不匹配 hillo 。
注意如果键太多可能会影响Redis的性能，所有最好使用精确查找，而不是使用*来进行查找。数据量庞大建议使用set集合代替。
当然我现在的数据库并没有任何的key:
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-10.jpg)
2. exists命令：*检查是否存在某个键*
`exists 键名`：若 key 存在，返回 1 ，否则返回 0 。
3. `move key db`：*移动key到db数据库*
如`move k1 5`：将k1移到5号库。
exists和move的测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-11.jpg)
4. `expire key seconds`：*设置key的过期时间*
key为键名，seconds为秒数，填写数字。
5. `ttl key`：*查看key还有多少秒过期。*
一般数字如50：表示还有50秒过期
-1表示永不过期，-2表示已过期或者key不存在。
tt1是time to live 的缩写。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-12.jpg)
过期则直接将该key删除。
6. `type key`：*查看key所存储的数据类型*
7. `del key`：*删除key*
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-13.jpg)

---
## 3.使用Jdis连接Redis并操作key命令
**1)新建Maven项目并配置pom.xml**
1. 新建maven项目。
使用一般模式创建，使用quick那个模板。
创建好的是没有src/main/resource目录和src/test/resource的，手动添加这两个目录。
2. 配置pom.xml，引入jedis的jar包。
可以在阿里的仓库中查找：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-14.jpg)
依赖为：
		<dependency>
		  <groupId>redis.clients</groupId>
		  <artifactId>jedis</artifactId>
		  <version>2.9.0</version>
		</dependency>
将该依赖添加到maven的pom.xml文件中。
配置好的pom文件（很多都是自动创建好的）：
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>Redis</groupId>
		  <artifactId>redistest</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  <packaging>jar</packaging>
		
		  <name>redistest</name>
		  <url>http://maven.apache.org</url>
		
		  <properties>
		    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		  </properties>
		
		  <dependencies>
		  	<dependency>
			  <groupId>redis.clients</groupId>
			  <artifactId>jedis</artifactId>
			  <version>2.9.0</version>
			</dependency>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>4.12</version>
		      <scope>test</scope>
		    </dependency>
		  </dependencies>
		</project>

**2)使用jedis操作相关的数据：**
1. 创建相关的代码包以及测试包，在下面创建两个class文件,并创建一个基础操作类用于连接Redis服务，其他的代码继承至这个类：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-15-2.jpg)
2. BaseOperation的代码如下（如何使用jedis连接redis）：
		package Base;
		
		import redis.clients.jedis.Jedis;
		
		public class BaseOperation {
			public Jedis getRedis(){
				//连接Jedis服务器
				Jedis jed =new Jedis("10.186.151.205", 6379);
				//输入密码，否则只能登陆而不能获取数据
				jed.auth("123456");
				return jed;
			}
		}
返回了一个Jedis的对象jed,而这个对象里有非常多的操作Redis的API方法。
3. 简单测试使用ping测试是否可用，在KeyOperation.java中添加测试方法：
		public String PingCmd(){
			Jedis jd=getJedis();
			try{
				return jd.ping();
			}finally{
				//一定要关闭
				jd.close();
			}
		}
在测试方法中添加：
		@Test
		public void testPingCmd(){
			String p=kop.PingCmd();
			Assert.assertEquals("PONG", p);
			System.out.println(p);
		}
测试通过，结果就不截图了。

**3)使用jedis操作其他命令：**
1. keys命令及flushALll（DB）命令：
KeyOperation.java中添加以下方法：
		public void systoutSet(Set<String> ss){
			//遍历输出key
			Iterator<String> it=ss.iterator();
			while(it.hasNext()){
				System.out.println(it.next());
			}
		}
		public void KeysCmd(){
			Jedis jd=getJedis();
			try{
				//相当于keys命令，返回值为Set<String>
				Set<String> ss=jd.keys("k*");
				System.out.println("开始时");
				//遍历输出key
				systoutSet(ss);
				jd.set("k1", "v1");
				jd.set("k2", "v2");
				Set<String> ss1=jd.keys("k*");
				System.out.println("set后");
				systoutSet(ss1);
				jd.flushAll();
			//  jd.flishDB();
				Set<String> ss2=jd.keys("k*");
				System.out.println("flush后");
				systoutSet(ss2);
			}finally{
				//一定要关闭
				jd.close();
			}
		}
添加测试方法：
		@Test
		public void testKeysCmd(){
			kop.KeysCmd();
			Assert.assertTrue(true);
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-16.jpg)
2. 测试move,exists等方法：
Jedis使用getDB()方法就可以查看当前在几号库中。
main中的代码：
		public void MoveCmd(){
			Jedis jd=getJedis();
			try{
				System.out.println(jd.getDB()+"号数据库是否存在k1?"+jd.exists("K1"));
				jd.set("K1","V1");
				System.out.println("添加后"+jd.getDB()+"号数据库是否存在k1?"+jd.exists("K1"));
				jd.move("k1", 5);
				System.out.println("移动后"+jd.getDB()+"号数据库是否存在k1?"+jd.exists("K1"));
				jd.select(5);
				System.out.println("移动后"+jd.getDB()+"号数据库是否存在k1?"+jd.exists("K1"));
				jd.del("K1");
				System.out.println("删除后"+jd.getDB()+"号数据库是否存在k1?"+jd.exists("K1"));
				jd.select(0);
			}finally{
				//一定要关闭
				jd.close();
			}
		}
测试类中的代码：
		@Test
		public void testMoveCmd(){
			kop.MoveCmd();
			Assert.assertTrue(true);
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-17.jpg)
3. expire，ttl,type等的Jedis方法名和命令一模一样。
使用方式也一模一样。就不多测试了。

---
## 4.Redis五种基本数据类型之String字符串
**1)关于Redis的5种基本类型**
1. redis是以键值对来存储数据的，键都是String类型(也可以存二进制byte)
值则主要有五种类型：
String（字符串），Hash(哈希表)，List(列表)，Set(集合)，SortedSet（有序集合）
2. String类型：
Redis中最基本的数据类型，可以存储任何形式的字符串，包括二进制数据，序列化之后的数据，Jsoh化后的数据甚至是一张图片。
3. 其余数据类型后面介绍。

**2)字符串类型的重要常用命令（重要）：**
1. 设置值:`set`
添加字符串值：`set key value`
如果 key 已经持有其他值， SET 就覆写旧值，无视类型。
对于某个原本带有生存时间（TTL）的键来说， 当 SET 命令成功在这个键上执行时， 这个键原有的 TTL 将被清除。
参数（2.6.12 版本以上）：
		EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。
		PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。
		NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
		XX ：只在键已经存在时，才对键进行设置操作。
2. 取值：`get`
获取对应key的字符串值：`get key`
如果key不存在那么返回特殊值 nil 。
假如key储存的值不是字符串类型，返回一个错误，因为 GET 只能用于处理字符串值。
3. `incr`:加1
`incr key`
将 key 中储存的数字值增一。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
4. `decr`:减1，和上面的相反
`decr key`
将 key 中储存的数字值减一。注意事项和上面相同。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-18-1.jpg)
5. `setex`：set expire的简写，设置值并设置key的过期时间
`setex key second value`
 将值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)。
如果 key 已经存在， SETEX 命令将覆写旧值。
这个操作是一个原子操作，不可分割。
该命令在 Redis 用作缓存时，非常实用。
6. `setnx`:set if not exist
`setnx key value`
只有在key中没有值时才可以存入，否则不进行任何操作。
7. `getset`：替换并返回旧值
`getset key value`
将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
当 key 存在但不是字符串类型时，返回一个错误。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-19.jpg)

**3)一些其他常用命令（更多命令查看手册）：**
1. `strlen`：返回所存储的字符串长度
`strlen key`
返回 key 所储存的字符串值的长度。
key不存在时返回0
2. `append`：加值
`append key value`
如果key已经存在并且是一个字符串，APPEND命令将value追加到key原来的值的末尾。
如果key不存在， APPEND 就简单地将给key设为value，就像执行 `SET key value`一样。
3. `incrby`：指定步长的增长
`incrby key n`:以每次加n来增加，只能对数值操作
如果key不存在，那么key的值会先被初始化为0，然后再执行INCRBY操作。
4. `decrby`：指定步长的减少
`decrby key n`:以每次加n来减少，只能对数值操作
其余和increby一样。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-20.jpg)
5. `getrange`：截取字符串
`getrange key start end`:截取key的值从start到end
负数偏移量表示从字符串最后开始计数，-1表示最后一个字符，-2表示倒数第二个，以此类推。
getrange通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。
字符串的下标从0开始。
6. `setrange`：替换指定的字符串
`setrange key start value`
将从start开始的和value一样长度字符串替换为value。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-21.jpg)
7. `mset`：同时设置一个或者多个键值对
`mset k1 v1 k2 v2 k3 v3...`
如果某个给定 key 已经存在，那么 MSET 会用新值覆盖原来的旧值，如果这不是你所希望的效果，请考虑使用`MSETNX`命令：它只会在所有给定key都不存在的情况下进行设置操作。
MSET是一个原子性(atomic)操作，所有给定 key 都会在同一时间内被设置，某些给定 key 被更新而另一些给定 key 没有改变的情况，不可能发生。
8. `mget`：同时获取一个或多个给定key的value
`mget k1 k2 k3...`
和上述`mset`注意事项相同
9. `msetnx`：同时设置一个或者多个键值对，不覆盖
`msetnx k1 v1 k2 v2 k3 v3...`
只要有一个键的值存在，则所有的key设置都会失败。
当所有key都成功设置，返回 1 。
如果所有给定key都设置失败(至少有一个key已经存在)，那么返回 0 。
测试后面三个方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-22.jpg)

---
## 5.使用Jedis操作字符串
**1)准备工作：**
1. 创建操作类继承至BaseOperation：
		package Operate;
		import Base.BaseOperation;
		public class StringOperation extends BaseOperation{
		}
2. 创建测试类：
		package Test;	
		public class RedisStringTest {
			StringOperation sop=new StringOperation();
		}

**2)创建操作方法及测试类：**
1. 测试一：
操作方法：
		public void Test01(){
			Jedis jd=getJedis();
			try{
				jd.set("k1", "1");
				System.out.println("get(key)="+jd.get("k1"));
				jd.append("k1", "23");
				System.out.println("append后get(key)="+jd.get("k1"));
				jd.incr("k1");
				System.out.println("incr后get(key)="+jd.get("k1"));
				jd.decrBy("k1", 100);
				System.out.println("减一百后get(key)="+jd.get("k1"));
				jd.flushAll();
			}finally{
				//一定要关闭
				jd.close();
			}
		}
测试方法：
		@Test
		public void testTest01(){
			sop.Test01();
			Assert.assertTrue(true);
		}
结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-23.jpg)

2. 测试二：
操作方法：
		public void sysoutList(List<String> ls){
			for (String s : ls) {
				System.out.print(s+" ");
			}
			System.out.println();
		}
		public void Test02(){
			Jedis jd=getJedis();
			try{
				jd.mset("k1","v1","k2","v2","k3","v3");
				System.out.print("初始时获取的k1,k2,k3值为：  ");
				List<String> lsList=jd.mget("k1","k2","k3");
				sysoutList(lsList);
				jd.mset("k1","111","k2","222","k3","333");
				System.out.print("mset覆盖，获取的k1,k2,k3值为：  ");
				sysoutList(jd.mget("k1","k2","k3"));
				jd.msetnx("k1","aaa","k2","bbb","k3","ccc");
				System.out.print("msetnx加值不覆盖,获取的k1,k2,k3值为：    ");
				sysoutList(jd.mget("k1","k2","k3"));
				jd.setex("k1", 5, "6666666");
				System.out.println("setex5秒内的取值"+jd.get("k1"));
				Thread.sleep(5100);
				System.out.println("setex5秒后的取值"+jd.get("k1"));
				jd.flushAll();
			}catch(InterruptedException e){
				e.printStackTrace();
			}finally{
				//一定要关闭
				jd.close();
			}
		}
测试方法：
		@Test
		public void testTest02(){
			sop.Test02();
			Assert.assertTrue(true);
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00034Redis%E5%AD%A6%E4%B9%A02-24.jpg)
3. 其余方法不测试了，使用方式和命令时一样的。

---
## 6.命令总结
**1)客户端常用命令：**
`ping`： 测试服务是否可用，如果返回pong则说明服务器正常
`select`： 切换库，如果要切换到n号库就是用`select n`
`flashall`： 删除所有库的数据
`flashdb`： 删除当前数据库内的数据
`config get *`： 获得redis的所有配置值
`exit`或者`quit`： 退出客户端cli
`dbsize`： 查看当前数据库中有多少个key
`info`： 查看Redis服务器的统计信息

**2)Key常用命令：**
`keys *`：列出该数据库中的所有键
`keys h?llo`： 匹配 hello ， hallo 和 hxllo 等。
`keys h*llo`： 匹配 hllo 和 heeeeello 等。
`keys h[ae]llo`： 匹配 hello 和 hallo ，但不匹配 hillo 。
`exists key`：若 key 存在，返回 1 ，否则返回 0 。
`move key db`： 移动key到db数据库
`move k1 5`： 将k1移到5号库。
`expire key seconds`： 设置key的过期时间
`ttl key`： 查看key还有多少秒过期。
`type key`： 查看key所存储的数据类型
`del key`： 删除key

**3)String类型常用命令：**
`set key value`： 添加(设置)字符串值
`get key`：获取key键对应的值
`incr key`： key的值加1
`decr key`： key的值减1
`setex key second value`： 添加一个second秒过期的值
`setnx key value`： 只有在key中没有值时才可以存入，否则不进行任何操作。
`getset key value`： 替换并返回旧值

`strlen key`： 返回key所存储的字符串的长度
`append key value`： 加值，新的值=旧值+value
`incrby key n`： 以每次加n来增加，只能对数值操作
`decrby key n`： 以每次加n来减少，只能对数值操作
`getrange key start end`： 截取key的值从start到end
`setrange`： 替换指定的字符串
`setrange key start value`： 将从start开始的和value一样长度字符串替换为value。
`mset k1 v1 k2 v2 k3 v3...`： 同时设置一个或者多个键值对
`mget k1 k2 k3...`：同时获取一个或多个给定key的value
`msetnx k1 v1 k2 v2 k3 v3...`：同时设置一个或者多个键值对，不覆盖

---
