---
title: MyBatis学习笔记（八）：Mybatis缓存配置
date: 2018-4-11 19:02:26
tags: MyBatis
categories: Java框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.缓存简介
1. 使用缓存可以使应用更快地获取数据，避免频繁的数据库交互，尤其是在查询越多，缓存命中率越高的情况下，使用缓存的作用就更加明显。
2. 一般提到MyBatis缓存时，指的都是二级缓存。
一级缓存，也叫本地缓存，系统会默认启用而且是不能控制的。
一级缓存也要了解，了解一级缓存可以避免产生一些难以发现的错误。

---
## 2.一级缓存测试
1. 创建测试类CacheTest.java,并在该类中创建测试一级缓存的testL1Chache方法：
		//测试一级缓存的代码
		@Test
		public void testL1Cache(){
			//获取sqlsession
			SqlSession sqlSession =getSqlSession();
			User user1 =null;
			try{
				//获取userMapper接口
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
				//调用seleceById方法，查询id=1的user
				user1=userMapper.selectByPrimaryKey(1L);
				//对当前对象重新赋值
				user1.setUserName("哈哈哈");
				//再次查询该用户
				User user2=userMapper.selectByPrimaryKey(1L);
				//查看着两者的名字
				System.out.println("user1name=  "+user1.getUserName());
				System.out.println("user2name=  "+user2.getUserName());
				//他们甚至完全就是同一个实例
				Assert.assertEquals(user1, user2);
			}finally{
				//关闭sqlsession
				sqlSession.close();
			}
			System.out.println("开启新的session");
			SqlSession sqlSession2 =getSqlSession();
			try{
				//获取userMapper接口
				UserMapper userMapper=sqlSession2.getMapper(UserMapper.class);
				//查询id为1014的用户,顺便测试一下id为1的
				User user2 =userMapper.selectByPrimaryKey(1L); //查到的结果是未修改之前的
				User user3 =userMapper.selectByPrimaryKey(1014L);
				System.out.println("user2name=  "+user2.getUserName());
				//执行删除操作
				int result=userMapper.deleteUserById(1014L);
				Assert.assertEquals(1, result);
				//删除成功后再次查询
				User user4 =userMapper.selectByPrimaryKey(1014L);
				//user3和user4是却不同的实例
				Assert.assertNotEquals(user3, user4);
			}finally{
				sqlSession2.rollback();
				sqlSession2.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-01.jpg)
2. 第一个SqlSession查询两次相同查询的对象相同。
查看日志则只查询了一次，第二次查询并未执行。
第二个SqlSession查询两次相同查询的对象却不同。
因为第二个SqlSession两次查询之间有一次对数据的改动操作。所以会进行两次查询。
3. MyBatis的以及缓存存在于SqlSession的生命周期中，在同一个SqlSession中查询时，MyBatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession执行方法和参数完全一致，通过算法会生成相同的键值，Map中已经有该键值存在，就会返回该对象。不会再次进行查询。
4. 有时为了避免这种错误可以选择关闭该方法的一级缓存：
		<select id="selectByPrimaryKey" flushCache="true" parameterType="java.lang.Long" resultMap="ResultMapWithBLOBs">
		</select>
在该对象的映射文件中配置该方法的flushCache属性为true就可以关闭一级缓存。
再次测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-02.jpg)
5. 注意，上面这样做会清除一级缓存，虽然解决了重复查询出相同的对象的奇怪错误，但是，这样会导致数据库连接增加，影响效率。

---
## 3.二级缓存的简介，配置及使用
**1)二级缓存简介：**
1. MyBatis的二级缓存不同于一级缓存，它存在与整个SqlSessionFactory的生命周期中。当存在多个SqlSessionFactory时,缓存都是绑定在各自的对象中的。只有在使用Redis这样的缓存数据库时，才可以共享数据库。
2. MyBatis的二级缓存可以使用默认的最贱但的配置，也可以集成Java自带的EhChache缓存，也可以集成Redis,memcached,scache,caffeine等缓存。

### 配置二级缓存
**1)默认开启的二级缓存：**
1. MyBatis全局配置settings中有一个参数cacheEnabled,这个参数是二级缓存的全局开关，默认值是true。不必过多配置，如果设置为false则后面的二级缓存就都不可用了。默认配置如下：
		<settings>
			<!--其他配置-->
			<setting name="cacheEnabled" value="true"/>
		</settings>
2. MyBatis的二级缓存是和命名空间绑定的，即二级缓存需要配置在Mapper.xml文件中或者配置在Mapper.java接口中。
映射文件中，命名空间就是xml根节点mapper的namespace属性。
在Mapper接口中，映射文件就是接口的全限定名称。

**2)Mapper.xml配置二级缓存：**
1. 最简单的配置方式，核心配置文件缓存开启的情况下，配置一个cache标签就可以了：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<mapper namespace="Mybatis.simple.mapper.RoleMapper">
		  <cache></cache>
		  <!--其他配置-->
		</mapper>
2. 上述配置的二级缓存默认相当于：
		<cache
			eviction="LRU"
			<！--no flushInterval-->
			size="1024"
			readOnly="false"></cache>
有以下属性：
	>映射语句中所有的Select语句会被缓存。
	映射语句文件中的Insert,Update,Delete语句会刷新缓存。
	缓存会使用LRU算法(最近最少未使用)来回收。
	根据时间表来自动刷新默认为no flushInterval，则不会以任何时间顺序刷新。
	缓存会存储集合或者对象的1024个引用。
	缓存时可读写的（read/write）,意味着对象检索不是共享的，安全，不存在其他程序的潜在修改。
3. cache标签各属性详解：
- `eviction`：回收策略，有下列选项：
	- `LRU`（最近最少使用）：移除最久未使用的对象，默认值。
	- `FIFO`（先进先出）：按对象进入缓存的顺序来移除他们。
	- `SOFT`（软引用）：移除基于垃圾回收器状态和软引用规则的对象。
	- `WEAK`(弱引用)：更积极移除基于垃圾回收器状态和弱引用规则的对象。
- `flushInterval`：刷新间隔。
可被设置为任意的正整数，代表一个合理的毫秒形式的时间段进行刷新缓存。
默认不设置，没有刷新时间间隔，缓存仅仅在调用SQL语句时刷新。
- `size`：引用数目。
可以被设置为任意正整数，默认为1024,要记住缓存的对象数目和运行环境可调用内存资源的数目。
- `readonly`：只读，可选值true/false
默认为false,可读可写。
只读方式返回的是缓存中的对象本身，速度更快，但是不允许修改对象，也不安全。
读写方式则是以序列化的形式返回了一个对象的拷贝，可修改，而且很安全，就是速度没有那么快。

**3)只使用注解方式时配置二级缓存：**
1. 在使用注解方式时，如果想对注解方式启用二级缓存，还需要在Mapper接口中进行配置。如果Mapper接口也存在对应的XML映射文件，两者同时开启缓存，则还需要另外的配置。
2. 只使用注解方式时只需要在接口上简单的添加注解即可：
		@CacheNamespace
		public interface RoleMapper {
			//各注解方式的接口方法，不含xml方式的方法。
		}
默认的配置和xml相同，但是配置的方式（属性）不同。
3. 也可以使用属性来具体配置：
		@CacheNamespace(
			eviction=FifoCache.class,
			flushInterval=60000,
			size=1024,
			readWrite=true
		)
		public interface RoleMapper {
			//各注解方式的接口方法，不含xml方式的方法。
		}
注意eviction需要使用.class方式，而且配置的是readWrite，不是readonly。

**4)同时使用xml和注解方式，需要使用CacheNamespaceRef**
1. 如果同时配置了注解及Xml:
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<mapper namespace="Mybatis.simple.mapper.RoleMapper">
		<cache
			eviction="LRU"
			flushInterval="60000"
			size="512"
			readOnly="false">
		</cache>
		</mapper>
注解配置如上，则会报错。
两者应该同时配置，而且配置应当相同。
2. 此时需要如下配置：（使用参照缓存）
有两种方式：
	- 在接口中使用参照缓存：
	RoleMapper.xml的配置不变，修改接口的配置：
			@CacheNamespaceRef(RoleMapper.class)
			public interface RoleMapper {
				//各注解方式的接口方法，不含xml方式的方法。
			}
	这样配置接口引用xml的缓存配置。
	- 在xml中配置参照缓存：
	接口的配置不变，将xml的配置修改如下：
			<cache-ref namespace="Mybatis.simple.mapper.RoleMapper" />
	这样配置xml引用接口的缓存配置。
3. 参照缓存的主要作用：
主要作用是为了解决脏读的问题，因为一般情况下很少出现xml和注解一同出现的情况。

### 使用默认的二级缓存
1. 使用前提：
因为可读可写的二级缓存是通过序列化来返回一个拷贝对象的，所以会使用到SerializedCache序列化缓存，这个这个缓存要求所有的被序列化的对象都要实现Serializable接口，否则会报错：
		org.apache.ibatis.cache.CacheException: Error serializing object.  Cause: java.io.NotSerializableException: Mybatis.simple.model.Role**
修改User类如下：
		public class User implements Serializable{
			private static final long serialVersionID=498465168L;  //值随便写
			//其他属性和getter,setter方法
		}
2. 编写测试类：
		@Test
		public void testL2Cache(){
			//获取sqlsession
			SqlSession sqlSession =getSqlSession();
			User user1 =null;
			try{
				//获取userMapper接口
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
				//调用seleceById方法，查询id=1014的user
				user1=userMapper.selectByPrimaryKey(1014L);
				//对当前对象重新赋值
				user1.setUserName("哈哈哈");
				//再次查询该用户
				User user2=userMapper.selectByPrimaryKey(1014L);
				//查看着两者的名字
				System.out.println("user1name=  "+user1.getUserName());
				System.out.println("user2name=  "+user2.getUserName());
				//user1，user2是同一个对象，因为在使用一级缓存
				Assert.assertEquals(user1, user2);
			}finally{
				sqlSession.close();  //此时才会将数据存到二级缓存中
			}
			System.out.println("开启新的session，后面开始使用二级缓存");
			SqlSession sqlSession2 =getSqlSession();
			try{
				//获取userMapper接口
				UserMapper userMapper=sqlSession2.getMapper(UserMapper.class);
				//再次查询id=1014的user
				User user3=userMapper.selectByPrimaryKey(1014L);
				System.out.println("user3name=  "+user3.getUserName());
				//user3和user1不是同一个对象
				Assert.assertNotEquals(user1, user3);
				//再次查询
				User user4=userMapper.selectByPrimaryKey(1014L);
				System.out.println("user4name=  "+user4.getUserName());
				//user3和user4不是同一个对象，但是值是相同的
				Assert.assertNotEquals(user4, user3);
			}finally{
				sqlSession2.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-03.jpg)
3. 结果分析：
只有调用close方法关闭SqlSession时，SqlSession才会保存查询数据到二级缓存中。所以一开始的二级缓存命中率都是0。
使用时应该是有二级缓存先查找二级缓存，没有二级缓存就使用一级缓存，再没有就去数据库查询。可以看到sqlSession2中二级缓存命中率分别为1/3,2/4。
如果查询完user3后修改对象值，user4的值不和user3相同。
4. 脏数据的问题：
二级缓存也并没有解决脏数据的问题。只是因为加了一句没什么卵用的`user1.setUserName("哈哈哈");`,就导致一级缓存的值改变从而影响二级缓存。所以要尽量避免这种无意义的修改。
5. MyBatis提供的二级缓存时基于Map实现的内存缓存，以及可以满足基本的应用了。但是需要缓存大量数据时，不能仅仅通过提高内存来使用MyBatis的二级缓存，还可以集成EhCache，Redis等缓存架构来实现。

---
## 4.集成EhCache缓存
**1)EhCache简介：**
1. EhCache是一个纯粹的Java进程内的缓存框架，具有快速，精干等特点。
2. 主要特点如下：
	- 快速
	- 简单
	- 有多种缓存策略
	- 缓存数据有内存和磁盘两级，无需担心容量的问题
	- 缓存的数据会在虚拟机重启的过程中写入磁盘
	- 可以通过RMI,可插入API等方式进行分布式缓存
	- 具有缓存和缓存管理器的侦听接口
	- 支持多缓存管理器实例以及一个实例的多个缓存区域
3. MyBatis官方提供了ehcache的缓存插件，可以在以下地址找到：
<https://github.com/mybatis/ehcache-cache>

**2)配置EhCache.xml：**
1. 添加Ehcache的pom.xml依赖：
		<dependency>
			<groupId>org.mybatis.caches</groupId>
			<artifactId>mybatis-ehcache</artifactId>
			<version>1.0.3</version>
		</dependency>
2. 简单配置EhCache:
详细的配置例子：<http://www.ehcache.org/ehcache.xml>
在src/main/resources目录下新增ehcache.xml文件，内容如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
		   xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
		   updateCheck="false" monitoring="autodetect" dynamicConfig="true">
		   
		   <diskStore path="D:/cache"/>  
		   <defaultCache
		   		maxElementsInMemory="3000"
		   		eternal="false"
		   		copyOnRead="true"
		   		copyOnWrite="true"
		   		timeToIdleSeconds="3600"
		   		timeToLiveSeconds="3600"
		   		overflowToDisk="true"
		   		diskPersistent="true">
		   </defaultCache>
		</ehcache> 
详细的配置介绍可以查看博客：
[ehcache.xml简介](https://blog.csdn.net/perfecttech/article/details/39081897)
3. 书上只介绍了copyOnRead和copyOnWrite这两个重要属性。
`copyOnRead`：从缓存中读取数据时是返回对象的引用还是复制一个对象返回。默认情况下是false,指返回数据的引用。true则是返回一个复制的对象，可读可写缓存。
`copyOnWrite`：写入缓存是直接缓存对象的引用还是复制一个对象然后缓存。默认为false。
如果想使用可读可写的缓存，则将这两个属性都设置为true。

**3)修改映射文件的缓存配置：**
1. ehcache-cache提供了两个可选的缓存实现：
`org.mybatis.cache.ehcache.EhcacheCache`和`.../LoggingEhcache`
但是这两者在使用时没啥区别，都会输出命中率日志（前者会使用装饰模式添加输出日志的功能）
2. 默认的缓存是使用defaultCache的，一般使用的时候都会有一个以映射文件的命名空间命名的缓存来对应。在ehcache.xml中添加配置(比如UserMapper要使用缓存)：
		<cache
			name="mbg.model.User"
			maxElementsInMemory="3000"
			eternal="false"
			copyOnRead="true"
			copyOnWrite="true"
			timeToIdleSeconds="3600"
			timeToLiveSeconds="3600"
			overflowToDisk="true"
			diskPersistent="true" />
如果未配置则使用默认的缓存。
3. 修改配置xml映射文件：只需要一句话，将cache修改如下
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<mapper namespace="mbg.mapper.UserMapper">
			<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
			<!--其他配置-->
		</mapper>
4. 这样就配置完毕了。
运行上面的默认二级缓存的测试代码，结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-04.jpg)
查看设置的控件D:/cache,发现多出了几个缓存文件。
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-05.jpg)

---
## 5.集成Redis缓存
**1)Redis的简介和Windows下的安装：**
1. Redis是一个高性能的key-value数据库。
因为学习Redis时所安装的是Linux版本，为了方便学习还是在在Windows下也安装一个吧。使用Redis缓存首先得需要开启Redis服务。
2. 在windows下安装Redis
下载：
<http://pan.baidu.com/s/1dEMCa9z>
或者：
<https://github.com/MicrosoftArchive/redis/releases>
选择合适的版本下载并解压到合适的目录，解压后的文件如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-06.jpg)
这样就安装完毕了。
3. 打开Redis服务：
在解压目录下shift+右键-->在此处打开命令行，输入如下命令将Redis安装到本地服务：
		redis-server.exe --service-install redis.windows.conf --loglevel verbose 
启动服务：
		redis-server.exe  --service-start
停止服务：
		redis-server.exe  --service-stop
卸载服务：
		 redis-server.exe  --service-uninstall
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-07.jpg)
使用两种客户端连接一下，测试是否打开服务：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-08.jpg)
成功打开服务，这样就可以开始配置Redis缓存了。

**2)配置Redis缓存（以User为例）：**
1. 添加pom.xml依赖：
		<dependency>
			<groupId>org.mybatis.caches</groupId>
			<artifactId>mybatis-redis</artifactId>
			<version>1.0.0-beta2</version>
		</dependency>
2. 在src/main/resources目录下新增redis.properties文件，内容如下：
		host=localhost
		port=6379
		connectionTimeout=5000
		soTimeout=5000
		password=
		database=0
		clientName=
配置了IP,端口，超时时间等。
3. 修改UserMapper.xml中的缓存配置：
redis-cache值提供了一个Mybatis的缓存实现：`org.mybatis.caches.redis.RedisCache`,只要配置这个type就可以了：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<mapper namespace="mbg.mapper.UserMapper">
			<cache type="org.mybatis.caches.redis.RedisCache"></cache>
			<!--其他方法-->
		</mapper>
4. 还需要配置实体类继承至srializable接口(默认的已经配置好了)：
		public class User implements Serializable{
			private static final long serialVersionID=498465168L;  //值随便写
			//其他属性和getter,setter方法
		}

**3)测试Redis二级缓存的特点：**
1. 使用和上面测试相同的代码：
第一次运行：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-09.jpg)
第二次运行：
![](http://p5ki4lhmo.bkt.clouddn.com/00036MyBatis%E5%AD%A6%E4%B9%A08-10.jpg)
2. 出现上述情况的原因：
Redis的缓存时全局共享的。第一次运行先使用一级缓存，再使用二级缓存。所得到的user1和user2是同一个对象。第二次运行，命中率都为1.0，说明两次都是使用的二级缓存的拷贝对象，因为是可读可写，这两个对象不相同所以报错。
3. 需要部署分布式应用时，需要使用Redis这种共享缓存，将分布式应用连接到同一个服务器，

---
## 6.脏数据的产生和避免
1. 二级缓存虽然能提高应用的效率，减轻数据库服务器的压力，但是如果使用不当，很容易产生脏读。
我们需要知道脏数据是怎么产生的，也需要知道如何去避免脏数据。
2. 一般情况下，每个二级缓存都对应了一个命名空间，不同的Mapper的二级缓存不会互相影响。而在一般的项目中往往会出现多表查询的情况，这个多表查询的方法会放在某一个映射文件中，而对这些表的增删改则是在其他的映射文件中。
这样当其他表更新时不会刷新这个Mapper的缓存，就会产生脏读。
3. 解决这种脏读的方法就是使用**参照缓存**，将几个关联的ER表使用同一个二级缓存：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<mapper namespace="mbg.mapper.UserMapper">
			<cache-ref namespace="Mybatis.simple.mapper.RoleMapper" />
			<!--其他方法-->
		</mapper>
4. 即使使用Redis,只是在同一个Mapper的不同SessionFactory之间共享，不会再Mapper之间共享。
5. 但是在关联表非常多的情况下，参照缓存就没有任何意义了。

---
## 7.二级缓存的适用场景
1. 并不是什么情况下都能使用二级缓存，推荐在以下场景使用：
	- 以查询为主的应用中，只有少量的增，删，改操作
	- 绝大多数以单表操作存在，很少存在互相关联的情况
	- 按业务划分对表进行分组时，关联的表较少时。
2. 建议在业务层使用可控制的缓存来代替二级缓存。

---