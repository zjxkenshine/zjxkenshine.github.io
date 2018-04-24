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
## 2.Redis主从复制（master/slave）
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

### 一台机器上搭建主从环境(两种方式实现主从复制)
**1)配置实现主从复制：**
1. 一台机器上只装了一个Redis也是可以模拟多服务器的请款的。可以创建多个Redis实例，就好比登好几个QQ那样。
假设需要搭建三个服务器，一个主两个从，可以进行如下操作。
2. 复制三份redis.conf用于测试并分别改名：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-12.jpg)
置空这三个文件：(使用数据流重定向)
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-13.jpg)
简单的配置方案(视频提供)：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-11.jpg)
3. 主服务器的配置文件：(6380端口)
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-14.jpg)
		include /usr/local/redis-4.0.9/redis.conf	-- 引入源配置
		daemonize yes	-- 开启默认后台启动，就不用使用&了
		port 6380	-- 启动端口
		pidfile /var/run/redis_6380.pid -- 进程号
		logfile 6380.log	-- 输出日志，也不用加nohup了
		dbfilename dump6380.rdb		-- rdb持久化文件名
从服务器的配置文件：(6381，6382端口)
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-15.jpg)
		...
		slaveof 127.0.0.1 6380		-- 表示是6380的从服务器
6382端口的配置和这个一样。
4. 启动三个端口服务：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-16.jpg)
登录客户端：`redis-cli -p 端口`
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-17.jpg)
再使用`info replication`查看状态。
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-18.jpg)
发现出现了：
`master_link_status:down`的状况。这表明失败了，up为成功，解决方法如下。
5. 上述问题的解决：
查看从服务器的输出日志发现如下错误：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-19.jpg)
`NOAUTH Authentication required`
百度了一下发现是主从服务器都设置了密码的问题（也可能是Redis版本不一致）。
解决方法：
关闭所有的客户端及服务。
在从服务器中加上如下配置：
		masterauth 123456
使其自动验证主服务器的密码（我的是123456）。
重新启动服务并使用`info replication`查看：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-20.jpg)
成功！！
6. 测试并验证组从复制的效果：
主服务器写入数据：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-21.jpg)
从从服务器得到数据：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-22.jpg)
从服务器写入数据报错：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-23.jpg)

**2)启动时指定主从关系**
1. 不重点介绍，需要将配置好的从配置文件中的
		slaveof 127.0.0.1 6380
删除。
2. 主服务器的启动相同。
然后在启动6381或6382端口时这样启动：
		`redis-server --slaveof 127.0.0.1 6380 redis6381.conf`
这样就指定了主从关系。
其余操作和上面的一样。

### 容灾处理
**1)容灾处理:**
1. 当master出现故障挂掉的时候，需要手动配置，将某一slave升级为master。
剩下的slave挂至新的master上(冷处理)。
2. 如上述测试的6380端口的机器宕掉了。
将6381升级为主服务器，可以修改配置文件，也可以：
在6381客户端下执行：
		slaveof no one		-- 设置为不是任何服务器的从(主)
在6382客户端下执行：
		slaveof 127.0.0.1 6381		-- 挂至6381端口
最好也将这两个端口配置的密码项修改一下(主的可以删除掉了)。
3. 然后6380的故障修复之后，再将6380挂到6381端口，并手动配置密码项：
		slaveof 127.0.0.1 6381
4. 冷处理：表示等主服务器凉凉了之后再处理。
热处理：出现故障时自动处理，自动恢复。
5. 注意：
**不配置时默认启动的服务器都是主服务器，master.**

**2）主从复制的总结：**
1. 一个master可以有多个slave
2. slave下线，读请求的处理性能下降
3. master下线，写请求无法执行
4. 当master发生故障，需手动将一台从服务器使用`slaveof no one`使其转换为主服务器。而其他从服务器则需要使用`slaveof ip port`来连接到新的master。
5. 若要实现自动化处理故障，需要使用到哨兵(Sentine)。
6. 关闭某一端口的redis：使用kill或者：
`redis-cli -p 6381 -a "123456" shutdown`

---
## 3.高可用哨兵(Sentine)
**1）哨兵简介：**
1. Sentine哨兵，守卫站岗，24小时不间断服务。
2. 哨兵时Redis官方提供的高可用方案，可以用它来监控多个Redis服务实例的运行情况。出问题时就会介入处理。
 
**2)哨兵的无密码简单配置及启动：**
1. 查看redis目录，可以看到有默认的哨兵配置文件：（sentinel.conf）
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-24.jpg)
在src目录下已经有了哨兵的可执行文件：（redis-sentinel）
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-25.jpg)
`redis-sentinel 哨兵配置文件`可以用于启动哨兵，一般情况下会启动多个哨兵，因为哨兵也有可能会出故障。（后面再详细说明怎么启动）
3. 复制多个哨兵配置文件(用于启动多个哨兵)：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-26.jpg)
打开这些个复制文件并执行以下修改：
1.将port端口修改为和哨兵文件名相对应。
2.找到并修改以下配置：
		sentinel monitor mymaster 127.0.0.1 6380 2
`mymaster`：一个名字可以随便修改
`127.0.0.1 6380`：监视Redis的ip，端口（**对应主服务器的地址**）
`2`：表示法庭人数，投票数量，一台服务器故障，监视这台服务器的哨兵需要另外的哨兵投票，投票数大于一半(2/3)，则认为真的挂了。
其实这三个配置文件都是用于监控主服务器的(6380端口)，虽然开启的端口和名字都不一样。
4. 启动哨兵：
指定配置文件来启动哨兵：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-27.jpg)
这样子是前台启动的，所以需要再建立两个链接去启动另外两个哨兵文件。
当然只要使用：
		# nohup redis-sentinel sentinel26380.conf &
就可以后台启动了，但是未指定日志文件，需要先使用nohup，启动效果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-28.jpg)
可以在一个Redis中启动多个哨兵进程。
5. 以上是主从服务器都没有密码是的配置，如果设置了密码，在哨兵处理故障时就不会处理(投票不通过)：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-30.jpg)
不会进行任何处理。
如果主从服务器设置了密码(最好一致)，则需要更多的哨兵配置(或者在宕机时手动处理)。

**3)哨兵的详细配置：**
1. 如果系统中使用了redis哨兵集群，由于在切换master的时候，原本的master可能变成slave，故也需要在原本redis master上配置masterauth：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-29.jpg)
2. 哨兵的配置：
		#sentinel端口
		port 26380
		#工作路径，其实使用默认的/tmp就可以，也可以自己配置
		dir "tmp"
		# 守护进程模式，不用使用&了
		daemonize yes
		# 指明日志文件名
		logfile sentinel26380.log
		#哨兵监控的master，主从配置一样，投票数>50%
		sentinel monitor mymaster 127.0.0.1 6380 2
		# master或slave多长时间（默认30秒）不能使用后标记为s_down状态。
		sentinel down-after-milliseconds mymaster 5000
		# 指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
		sentinel parallel-syncs mymaster 1
		#若sentinel在该配置值内未能完成failover操作（即故障时master/slave自动切换），则认为本次failover失败。
		sentinel failover-timeout mymaster 18000
		#设置master和slaves验证密码
		sentinel auth-pass mymaster 123456
清空并配置sentinel26380.conf如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-31-2.jpg)
清空并配置sentinel26381.conf如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-32-2.jpg)
sentinel26382.conf的配置和上面的相同：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-33-1.jpg)
所有的哨兵的`sentinel monitor mymaster 127.0.0.1 6380 2`配置必须是监视主服务器的。
3. 测试哨兵的故障转移：
首先开启三个Redis服务器对象及三个哨兵：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-34.jpg)
查看主从服务器是否异常：（up说明是正常的）
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-35.jpg)
关闭6380的主服务器，隔几十秒再打开6381的客户端查看主从关系：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-36.jpg)
发现6381的服务器已经变成了主服务器,在/tmp中查看6380哨兵的日志如下(后半部分)：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-37.jpg)
发现自动执行了转换，重新开启6380的客户端并且查看主从关系：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-38.jpg)
可能会有延时，刚打开时会出现依旧是主服务器但是没有从服务器的情况，等一会儿在次查看就好了。最后查看一下哨兵的配置文件，会发现有很多自动添加的配置，其实启动哨兵之后就添加了，不用管它：
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-39.jpg)
最后关闭的时候如果要保持目前的主从设置记得要先关闭哨兵，再关闭Redis服务。
不过也没啥关系，哨兵都会帮我们处理。

**4)总结：**
1. 使用文件配置的方式需要在服务器挂掉之后进行手动的处理。手动修改主从关系。
使用起来挺麻烦，尤其是从服务器对象很多的时候。
2. 使用哨兵Sentine的方式虽然需要额外的一点配置，但是使用起来非常方便，推荐使用。
3. 使用客户端时要直接连接到sentinel的端口(随便哪个)，这样可以保证高可用性(前面直接连接到redis-server只是为了测试)：
		sed "s/6379/6380/g" redis-6379.conf > redis6381.conf

---
## 4.Redis的安全
**1)设置密码：**
1. 设置Redis的密码，在redis.conf文件配置如下就行了：
		requirepass 123456
2. 注意：
因为Redis的速度非常快，所以需要使用非常强大的密码。否则会被暴力破解。
此时的客户端连接就需要使用`auth 123456`或者`redis-cli -a "123456"`来连接了。

**2)绑定ip：**
将redis.conf文件中的`# bind 127.0.0.1`的注释去掉，并将这个ip修改为允许访问的ip,那么除了这个ip以外的其他ip都不允许访问。

**3)命令的禁止或重命名：**
主要是针对flushall和flushdb:
在Redis.conf中进行禁止或者重命名的配置：(一般在密码项的下方)
1. 重命名FLUSHALL命令：
rename-command FLUSHALL faegihegowiefohaegw..（反正是一大串不可能会出现的命令）
配置完之后就只能使用后面这一串命令来执行清除。
注意，需要在aof文件中没有FLUSHALL命令，否则服务器会启动失败。
其他命令的改名也需要如此。
2. 禁止FLUSHALL及FLUSHDB命令：
		rename-command FLUSHALL "" 	置空表示禁止
		rename-command FLUSHDB "" 	禁止FLUSHDB
3. 禁止config命令：
		rename-command config ""
config可以看到服务器的信息，所以需要禁止掉。
![](http://p5ki4lhmo.bkt.clouddn.com/00038Redis%E5%AD%A6%E4%B9%A05-40.jpg)

**4)x修改默认端口**
默认的端口为6379，最好修改一下。
修改port就可以了。

视频的课程学完啦！

---
## 5.课程总结
纸上得来终觉浅，绝知此事要躬行。
Redis,这些仅仅是个开始。

---