---
title: Redis学习笔记（一）：简介，下载安装，启动及客户端连接
date: 2018-04-05 21:44:34
tags: Redis
categories: NoSQL

---
## 0.学习资料
参考视频：《动力节点Redis视频教程》
参考网址：<http://www.redis.net.cn/tutorial/3501.html>

---
## 1.Redis介绍
**1)Redis及NoSQL简介** 
1. Redis简介：
 - Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
 - Remote Dictionary Sever,远程目录服务器，是一个使用C语言编写的key-value服务器。值可以使字符串，Map，列表（list）,集合(sets)和有序集合（store sets）等类型。通常也被称为数据结构服务器。

2. NoSQL数据库：
Redis属于NoSQL,非关系型数据库(NOT ONLY SQL)。传统的数据库MySQL,Orcale等都是关系型数据库。NoSQL在2009年以后才得到空前方法，主要是因为web2.0的发展，导致关系型数据库不太适用。
3. 各种不同存储方式的NoSQL：
	- key-value存储：
		- Berkeley DB
		- MemcacheDB：近几年使用减少，但仍有大量项目使用该数据库
		- Redis：目前的主流
	- 文档存储：
		- MongoDB
		- CouchDB
	- 列存储：
		- Hbase
		- Cassandra
4. Redis的发展历程简介：
2008年意大利一家创业公司基于Mysql开发了一个统计网站LLOOGG,该公司创始人，Redis之父Salvatore Sanfilippo对MySQL性能感感到失望，就开发了Redis。
同年将Redis开源并与另一位主要开发者Pieter NoordHuis开发至今。

**2)Redis的优点：**
1. Redis 与其他 key - value 缓存产品有以下三个特点：
	- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
	- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
	- Redis支持数据的备份，即master-slave模式的数据备份。
2. Redis的优势：
	- **性能极高** – Redis能读的速度是110000次/s,写的速度是81000次/s 。
	- **丰富的数据类型** – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
	- **原子** – Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
	- **丰富的特性** – Redis还支持 publish/subscribe, 通知, key 过期等等特性。
3. Redis相比其他key-valueNoSQL的优势：
	- Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
	- Redis运行在内存中可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，应为数据量不能大于硬件内存。在内存数据库方面的另一个优点是， 相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。 同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

---
## 2.Redis的下载及编译安装
**1)下载到Linux**
1. 下载：
官网：
<https://redis.io/>
中文网：
<http://www.redis.net.cn/>
github地址：
<https://github.com/antirez/redis>
或：
<https://github.com/dmajkic/redis/downloads>
2. 一般的Redis使用都是在Linux下的，官方版本也是Linux的。
Windows版本的是非官方的，由微软开发的，仅供学习使用,不能在正式项目中使用。
3. 将文件使用Xftp传送到用户主目录下：
因为可能会涉及到权限问题，所以使用root用户登录。
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-01-1.jpg)
4. 使用Xshell查看Redis是否传到了Linux
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-02.jpg)
下面就可以开始安装了


**2)安装Redis**
1. 解压：
使用如下命令：
		# tar -zxvf redis-4.0.9.tar.gz -C /usr/local/
运行测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-03.jpg)
东西很多，但是很快就解压好了。
然后跳转到/usr/local/,查看是否有redis：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-04.jpg)
2. 进入到redis目录进行编译：
cd到redis目录内，
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-05.jpg)
可以看到有个makefile文件，代表时C语言开发，需要编译。
使用make命令进行编译：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-06.jpg)
3. 常见编译问题一：找不到gcc
`error：gcc/cc not found`
由于是C语言，所以编译需要C语言的编译器：gcc,没有gcc编译器就会报错。
gcc是GUN complier collection的缩写，是Linux下的一个编译器集合，主要用于编译C和C++程序。
执行如下命令安装gcc：
		# yum -y install gcc
		-y：表示自动确认安装过程中的确认步骤。
由于我的Linux已经装了gcc，就不测试了，安装完毕后使用which命令查看到：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-07.jpg)
就说明安装成功了。
4. 注意，再次make之前最好使用以下命令清理一下上次错误编译遗留下来的文件：
		make distclean
5. 常见编译问题二：
		error:jemalloc/jemalloc.h No such file or dictionary
出错原因，没有找到.h文件，解决：
使用内存分配器libc替代,将编译代码改为：
		make MALLOC=libc
6. 编译过后的redis一般就可以使用了。
不过有时候还会执行`make install`命令。

**3)关于make install**
1. 编译完毕后的可执行文件默认放在`/usr/local/redis-4.0.9/src`目录下：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-08.jpg)
绿色的为可执行文件。
2. 执行`make install`命令后相当于将可执行文件复制放到PATH目录(usr/local/bin)中，执行起来可以方便一点。
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-09.jpg)

---
## 3.Redis的几种启动方式
**1)前台启动:**
1. 前台启动:
退出命令行后Redis服务就自动关闭了。
2. 执行了`make install`可以在任何地方使用下列命令启动：
		# redis-server
未执行`make install`则需要在src目录下使用：
		# ./redis-server
后面的介绍都是执行了`make install`的。
3. 启动测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-10.jpg)
使用Ctrl+C退出。
使用ps配合grep查看进程：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-11.jpg)
发现并没有redis-server的进程。
说明服务已经被关闭了。

**2)后台启动:**
1. 只要在启动时加&就可以了：
		# redis-server &
启动测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-12.jpg)
2. 退出并查看进程：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-13.jpg)
发现redis的进程依旧在。

**3)后台启动时输出日志到nohup.out文件:**
1. 只要在启动前加上nohup就可以了：
		# nohup redis-server &
nohup命令在执行其他程序时也经常使用。
执行时按Enter进行操作。
2. 执行完nohup后当前目录会出现一个nohup.out文件，可以使用cat查看。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-15.jpg)

---
## 4.Redis的两种关闭方式
主要针对的是后台启动的redis。
1. 强制关闭：
任何软件都可以这样关闭：
		kill 进程号
或者
		kill -9 进程号
直接关，不多bb，很粗暴，会导致数据错误
2. 使用redis自带的关闭方式：
		# redis-cli shutdown
		cli是redis的一个客户端
这种方式会等待当前正在执行的请求执行完毕再关闭。比较温和，推荐使用。
关闭测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-14.jpg)
3. 注意，想要再次后台启动redis服务时一定要先将原来的redis服务关闭。

---
## 5.Redis的三种客户端的连接
### a.命令行客户端redis-cli
1. 介绍
redis-cli（Redis Command Line Interface）是redis自带的基于命令行的客户端，用于与服务器的交互。可以使用该客户端来执行各种redis的命令。
2. 连接方式一：直接连接
		# redis-cli
默认ip为127.0.0.1，端口为6379
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-16.jpg)
只装了一个redis默认的就是127.0.0.1和6379.
3. 连接方式二：指定ip和端口
		# redis-cli -h ip -p 端口
默认的就是这样连接
		# redis-cli -127.0.0.1 ip -p 6379
4. 退出客户端/关闭连接：
使用【Ctrl】+C或者输入exit/quit就可以退出了。

### b.远程客户端
用于在远程图形界面连接redis。
如xshell,xftp就是远程连接Linux的一种客户端。
常用的redis远程客户端是Redis DeskTop Manager。
**1)Redis DeskTop Manager下载安装**
1. 常见的客户端，但是功能不是很强。
下载地址：
<https://redisdesktop.com/download>
我下载的是windows版本的客户端。
下载完之后直接按提示安装就行了，很简单的。
2. 可能会安装VC++环境，不用管等它安装完毕就可以了，安装完毕后打开是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-17.jpg)


**2)配置Redis：**
1. 远程连接前需要配置（授权）：是修改Redis主目录配置文件
在Xshell中cd到Redis主目录，可以看到有个名为redis.conf的配置文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-05.jpg)
2. 使用vim打开redis.conf文件：
		# vim redis.conf
可以看到有很多蓝色注释的地方。
首先将69行左右的`bind 127.0.0.1`使用`#`注释掉,它指定了只能本地连接：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-18.jpg)
3. 重启redis并指定修改过后的config配置文件启动：
先关闭，然后启动的命令如下（在Redis目录下）：
		# nohup redis-server redis.conf &
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-20.jpg)
4. 设置密码：
使用`# redis-cli`进入命令行客户端后使用`config set requirepass 密码`设置密码，再次登录并使用`auth 密码`登录
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-19-1.jpg)
5. 在Windows的cmd中使用telnet测试端口：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-21.jpg)
能连通的话就可以了。
不能连通的话在xsehll下执行如下命令：
		firewall-cmd --query-port=6379/tcp
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-22.jpg)
如果是no则使用以下命令开启：
		firewall-cmd --add-port=6379/tcp
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-23.jpg)
6. 再次连接`telnet 10.186.151.205 6379`，发现能进入环境：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-24.jpg)

**3)连接Redis**
1. 点击连接到服务器新建连接并配置如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-25.jpg)
2. 点击左下角测试连接，如图，测试成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-26.jpg)
3. 然后点击OK连接成功，如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00031Redis%E5%AD%A6%E4%B9%A01-27.jpg)

**4)phpRedisAdmin**
一个php的客户端，需要php的运行环境：
下载建站继承软件包：xampp
具体使用过程以后遇到再百度吧。

### c.编程客户端
与开发密切相关，主要输Java的客户端。
**1)编程客户端简介：**
redis是以键值对存储数据在服务器上，为了使java能够读取键值对内的数据，有大佬就编写了专门的api，就像驱动程序一样，使用这个Api就可以访问服务器上的各种数据并对数据进行操作。
**2)Redis的Java客户端：**
1. Jedis:
Redis官方推荐的java客户端，很小但是很全面。
Jedis兼容Redis的各种版本。
github地址：<https://github.com/xetorthio/jedis>
API文档：http://tool.oschina.net/uploads/apidocs/
具体的使用以后会重点介绍。
2. Lettuce:
国外用的比较多，是一个可伸缩线程安全的Redis客户端，多个线程可以共用一个RedisConnection,利用优秀的netty nio来高效地管理多个连接。
源码：<https://github.com/lettuce-io/lettuce-core>
3. 其余还支持40余种各种语言的Redis客户端。

---