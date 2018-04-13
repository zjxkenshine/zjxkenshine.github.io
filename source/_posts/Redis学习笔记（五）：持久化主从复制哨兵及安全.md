---
title: Redis学习笔记（五）：持久化，主从复制，高可用哨兵及安全
date: 2018-04-13 17:25:56
tags: Redis
categories: NoSQL

---
## 0.参考资料：
参考视频：《动力节点Redis视频教程》
参考教程：<http://www.redis.net.cn/tutorial/3501.html>
Redis命令参考手册：<http://redisdoc.com/>
w3cschoolRedis教程：<https://www.w3cschool.cn/redis/nma27f21.html>

---
## 1.Redis持久化及两种实现方式
**持久化简介**
1. 持久化可以理解为存储，就是将数据存储到一个不会丢失的地方。
Redis把数据放在内存中，电脑关闭或者重启数据就会消失。所以不是一种持久化，而把Redis的数据放到磁盘里就是一种持久化。
2. Redis的数据存储在内存中，如果Linux宕机或者重启，或者Redis自己崩溃重启，那么数据就会消失。为了解决这个问题，Redis提供了两种机制对数据进行持久化存储--RDB方式和AOF方式。便于发生故障后迅速恢复数据。
3. AOF方式相当于时RDB方式的升级版。

### RDB方式实现持久化
**1)RDB方式简介：**
Redis Database的缩写，就是在指定的时间间隔内将数据集快照写入磁盘。
数据恢复时再将快照文件直接读入到内存中。**默认是开启的**，如果有特殊需求只要修改配置文件就可以了。

**2)RDB方式的实现：**
1. 只需要配置redis.conf文件即可。
2. 使用vim打开redis.conf文件，找到SNAPSHOTING快照：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-01.jpg)
3. SNAPSHOTING常用配置项简介：
		save 900 1
		save 300 10
		save 60 10000
`save second changes`表示在多少秒内有多少次变化则会自动备份。
默认900秒内若有一次改变就保存一次，300秒内有10次改变就保存一次，60秒内有10000次改变就保存一次。这些是可以自动保存的。
		dbfilename dump.rdb
RDB的文件名，默认文件名为dump.rdb。
		dir ./
RDB和AOF文件存储的路径。
查看dump.rdb:
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-02.jpg)

### AOF方式实现持久化
**1)AOF简介：**
Append-only File(AOF)：Redis每次接收到一个改变数据的命令时，就将该命令写入到AOF文件中。如果是读取数据的命令则不会写入。在Redis重启时，只要执行AOF中的所有命令就可以恢复数据了。

**2)AOF的实现方式：**
1. 也是只需要配置redis.conf文件就可以了。
2. 找到APPEND ONLY MODE这一项：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-03.jpg)
这就是AOF的配置项。默认是关闭的。
将appendonly改为yes就是开启aof。
3. 常用的配置项介绍：
		appendonly no
AOF方式的总开关。
		appendfilename "appendonly.aof"
AOF文件的文件名。
保存路径和RDB共用dir。

**3）其它常用配置项介绍：** 
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-09.jpg)
1. 关于保存策略：
		# appendfsync always
		appendfsync everysec
		# appendfsync no
配置向aof文件写命令数据的策略：
有三种策略：
`no`：不主动进行持久化同步，由操作系统进行同步，比较快但是不是很安全。
`always`：每次执行写命令都执行同步，安全但是很慢。
`everysec`：没秒执行一次，介于安全和速度之间，默认值。
2. 关于重写策略的两个配置（重要）：
		auto-aof-rewrite-percentage 100
		auto-aof-rewrite-min-size 64mb
(重写：去除一些无用的数据，冗余，对命令进行优化，减少文件的体积)
上面的配置表示当目前的aof文件超过上一次重写aof文件的大小的百分之多少是进行重写。如果之前未重写则以启动时的aof大小为依据
下面表示允许重写的最小aof文件的大小：达到64mb才开始整理。一般线上的时候是配置为好几GB。

**4)测试：**
先重启Redis并指定使用redis.conf文件开服务:
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-04.jpg)
打开客户端，进行多个数据修改命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-05.jpg)
重启Redis并打开查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-06.jpg)
数据并未丢失。查看文件列表，发现多了个aof文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-07.jpg)
可以看内部的文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-08.jpg)

**5)AOF方式总结：**
1. aof方式是一种可以完整实现数据保存的解决方案。
2. aof文件会在操作过程中变的越来越大，但是Redis支持在不影响服务的情况下对aof文件中进行重写。
3. 可以同时使用aof方式和rdb方式，Redis优先加载aof方式的文件。

---
## 2.Redis集群之主从复制
**1)主从(master/slave)复制简介：**
1. 通过持久化功能，Redis保证了在服务器重启的情况下数据也不会丢失（或少量丢失），但是由于数据时存储在一台服务器上的，那么这台服务器出现故障，比如硬盘坏了，那么也会导致数据丢失。
2. 为了避免单点故障，就需要将数据复制多份部署到不同的服务器上，即使有一台服务器出现故障，其他服务器也能继续提供服务。
3. 要求当一台服务器数据更新以后，自动将更新的数据同步备份到其他服务器上，者就需要用主从复制实现了。

**2)主从复制的实现：**
1. Redis提供了复制（replication）功能来自动实现多台Redis服务器的数据同步。
主从架构示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-10.jpg)
2. 实现原理：
可以通过部署多台Redis,并在配置文件中指定这几台Redis之间的主从关系，主负责写入数据，同时把写入的数据同步到从Redis,这种模式就叫做主从复制（master/slave）。
Redis默认的master用于写，slave用于读，向slave写数据会导致错误。
3. 有两种方式可以实现：
	- 方式一：修改配置文件，启动时，服务器读取配置文件，并自动成为指定服务器的从Redis,从而构成主从服务架构。
	- 方式二：`redis-server --slaveof <master -ip> <master -port>`,在启动Redis时指定当前服务器成为某一主服务器的slave,从而实现主从复制。
4. 方式一，修改配置文件：
简单的配置(就看看)：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-11.jpg)



---