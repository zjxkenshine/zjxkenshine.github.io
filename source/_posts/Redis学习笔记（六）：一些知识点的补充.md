---
title: Redis学习笔记（六）：一些知识点的复习及补充
date: 2018-4-23 12:35:49 
tags: Redis
categories: NoSQL

---
## 0.参考资料：
参考视频：慕课网教程《Redis从入门到高可用》
参考教程：<http://www.redis.net.cn/tutorial/3501.html>
Redis命令参考手册：<http://redisdoc.com/>
w3cschoolRedis教程：<https://www.w3cschool.cn/redis/nma27f21.html>

记录一些动力节点的视频未涉及到的知识点。

---
## 1.Redis基础知识的补充
**1)基础补充：**
1. 五种基本数据结构的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-01.jpg)
2. Redis的特点：
	- 速度快
	- 持久化
	- 多数据结构(5种基本+扩展)
	- 支持多种客户端
	- 功能丰富
	- 简单
	- 主从复制
	- 高可用(Sentinel哨兵)
	- 分布式(Redis Cluster)
3. 典型应用场景：
	- 缓存系统
	- 计数器
	- 消息队列系统
	- 排行榜(ZSET有序集合)
	- 社交网络(计数器的功能)
	- 实时系统(位图，过滤垃圾邮件等)

**2)Redis特点补充：**
1. Redis速度快的原因：
	- 数据存储在内存中，读写非常快。(主要原因)
	- 使用C语言编写，离操作系统层近，而且代码水平高。
	- 使用单线程-->内存模型中使用非常快。
2. 计算机介质的速度图示：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-02.jpg)
内存与硬盘的对比：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-03.jpg)
3. Redis的扩展数据结构：
	- BitMap：位图，用很小的内存实现高效存储。
	- HyperLogLog：使用非常小的内存(12k)来统计基数。
	- GEO：Redis3.2，地理信息位置的定位。
4. Redis的多功能简介：
	- 发布订阅
	- 事务
	- Lua脚本
	- pipeline
5. Redis的简单的体现：
	- 单机Redis只有23000行代码(集群分布式50000行)
	- 不依赖外部库
	- 单线程模型(开发容易)
6. 主从复制示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-04.jpg)

**3)Redis的安装及启动的补充：**
1. Linux下Redis的安装：
	- 下载：`# wget http://download.redis.io/releases/redis-4.0.9.tar.gz`
	- 解压：`# tar xzf redis-4.0.9.tar.gz`
	- 设置软连接（方便升级）：`# ln -s redis-4.0.9 redis`
	- 进入软连接目录：`# cd redis`
	- 编译：`# make`
	- 安装：`# make install`
2. 可执行文件的说明：
	- redis-server：redis服务器
	- redis-cli：redis命令行客户端
	- redis-benchmark：redis性能测试工具
	- redis-check-aof：AOF文件检测修复工具
	- redis-check-rdb：RDB文件检测修复工具
	- redis-sentinel：高可用哨兵工具
3. redis的三种启动方式：
	- 最简启动：`# redis-server`
	- 动态参数启动：`# redis-server --port 端口`
	- 配置文件启动：`# redis-server 配置文件`
4. 验证是否启动：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-05.jpg)
5. 三种启动方式的区别：
	- 生产环境选择配置文件启动
	- 单机多实例配置文件可以使用端口区分开
6. 启动客户端：
	- 本地Redis：`# redis-cli`
	- 指定ip及端口：`# redis-cli -h ip -p port`
	- 有密码：`# redis-cli -h ip -p port -a "密码"`
7. 客户端返回值:
	- 状态回复：PING-->PONG
	- 错误回复：(error)....
	- 整数回复：(integer) 1
	- 字符串回复："hello"
	- 多行字符串回复：执行mget的返回值等

**4)Redis的常用配置：**
1. Redis的常用配置:redis.conf中的配置
	- daemonize：是否以守护线程方式(后台方式)启动（默认是no,建议使用yes）
	- port：端口，默认6379
	- logfile：日志输出文件
	- dir：redis的工作目录
2. 可以在客户端使用`config get *`或者直接访问redis.conf文件来查看配置。
3. 还有很多的配置，后面章节会介绍：
	- AOF的配置
	- RDB的配置
	- 慢查询的配置
	- 最大内存的配置
	。。。
4. 查看所有配置的方式：(去注释,去空格)
		# cat redis.conf| grep -v "#" | grep -v "^$" 
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-06.jpg)
5. 将上述结果重定向到另一个配置文件则可以方便配置,如：
		# cat redis.conf| grep -v "#" | grep -v "^$" > redis6666.conf

---
## 2.Redis API使用和理解
通用命令以及五种基本数据结构的操作
### 通用命令，数据结构编码及单线程架构
**1)通用命令和内部编码：**
1. 通用命令：
	- keys：遍历该数据库的所有键
	- dbsize：数据库大小(key的总数)
	- exists key：key是否存在
	- del key [key,key...]：删除key
	- expire key：设置key的过期时间（second级）
	- ttl key：查看key的过期时间(-1：永久存在；-2或0：不存在)
	- persist key：去掉key的过期时间
	- type key：key的类型(none:不存在)
2. 注意事项：
	- keys命令一般不在生成环境中使用，会比较慢。时间复杂度为O(n)。
	- 生产环境一般使用scan命令。
	- keys命令可以在从节点(slave)使用。
	- dbsize，exists key的复杂度为O(1),可在生产环境使用。
3. 各命令的时间复杂度：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-07.jpg)
4. 5种基本数据结构内部编码：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-08.jpg)
redis内部数据结构：redisObject
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-09.jpg)
用户可以不知道编码方式的改变，只需要知道如何使用基本类型就可以了。

**2)单线程架构：**
1. 单线程架构：
串行执行：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-10.jpg)
在一个瞬间只会执行一条命令。
2. 为什么Redis的单线程这么快：
	- 纯内存(主要原因）
	- 非阻塞I/O:epoll实现IO多路复用，并且将epoll的读写，连接关闭转换为自身的事件来提高读写速度。
	![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-11.jpg)
	- 避免线程切换和竞态消耗。
3. 单线程的表现：
	- 一次只执行一条命令。
	- 拒绝执行长(慢)的命令。
	- 执行某些操作时会额外产生一个线程来执行：
	`fysnc file dicriptor`,`close file dicriptor`等

### 五种基本数据结构
**1)字符串：**
1. 字符串的value不止可以存字符串，还可以是以下的类型，会自动转换：
	- 整型
	- 二进制
	- json
2. 字符串value的最大限制：512MB
一般的键值对建议在100K以内。(需要考虑一些二级数据结构)
3. 使用场景：
	- 缓存
	- 计数器
	- 分布式锁
4. 常用命令及时间复杂度：
	- get,set：时间复杂度O(1)
	- del：O(1)
	- incr,decr：自增，自减一，O(1)
	- incrby key k,decrby key k：自增自减k，O(1)
	- setnx：不存在才设置，O(1)
	- setxx：key存在才设置，O(1)
	- mget，mset：批量的原子操作。O(n)
5. 其他命令：
	- getset：设置新值并返回老值。O(1)
	- append：追加值。O(1)
	- strlen：返回字符串长度，注意中文。O(1)
	- incrbyfloat：浮点型自增。O(1)
	- getrange key start end：获取指定下标的值。O(1)
	- setrange key index value：设置指定下标所对应的值。O(1)
6. incr id，计数器：
	- 统计数量
	- 缓存信息
	- 分布式id生成的简单策略
7. 重要命令的时间复杂度：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-12.jpg)

**2)hash哈希键值结构：**
1. 基本结构：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-13.jpg)
key相当于一个表，而field则是一个String类型的字段，value是String类型的值。
2. 常用方法：
	- hget,hset,hdel filed value：获取，设置，删除field的值，O(1)。
	- hexist key field：key中是否存在field,O(1)。
	- hlen：key中的field数量，O(1)。
	- hmget,hmset key field1 [value1]...：批量获取，添加，O(n)。
	- hincr,hincrby：计数器，O(1)。
3. 其他方法：
	- hgetall key：返回该key所有的field-value,O(n)。O(n)的都慎用。
	- hvals：返回所有的值，values,O(n)。
	- hkeys：返回所有的fields,O(n)。
	- hsetnx：不覆盖设置，O(1)。
	- hincreby,hincrebyfloat：O(1)。
4. hash的内部编码：
	- hashtable：默认的编码
	- ziplist：达到某一个值时使用，会节省非常多的空间。
5. hash无法设置field的过期时间。
6. 重要命令的总结：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-14.jpg)

**3)list列表：**
1. 基本结构：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-15.jpg)
2. 特点：
有序，可重复，可从左右进行插入弹出。
3. 重要命令：
	- lpush,rpush：从左边，右边添加,O(1~n)。
	- lpop,rpop：从左边，右边删除并返回值，O(1~n)。
	- linsert key before|after oldvalue newvalue：插入，O(n)。
	- lrem key count value：删除特定数量的重复元素，O(n)。
	- ltrim key begin end：保留索引范围的值，O(n)。
	- lrange key begin end：查询索引范围中的列表，O(n)。
	- lindex：按指定索引获取元素，O(n)。
	- llen：获取列表长度，O(1)。
	- lset key index newValue：将指定索引的值修改，O(n)。
4. 不太常用的命令：
	- blpop,brpop：阻塞版的删除，O(1)
	- rpoplpush,brpoplpush：rpop+lpush。
5. 命令组合使用的口诀：
	- LPUSH+LPOP = Stack栈
	- RPUSH+RPOP = Queue队列
	- LPUSH+LTRIM = 固定数量的集合
	- LPUSH+BRPOP = 消息队列

**4)Set集合：**
1. 集合示意图(模型)：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-16.jpg)
2. 集合的特点：
	- 无序
	- 不重复
	- 支持集合间的操作：交，差，并集
3. 集合内的API：
	- sadd,srem：添加与删除命令。
	- scard：计算集合元素个数。
	- sismember：是否是元素。
	- srandmember：从集合中随机查出n个元素，不删除。
	- spop：从集合中随机弹出一个元素。
	- smembers：获取集合所有元素，无序，少用。
	- sscan：在生成环境中遍历集合。
4. 集合间的API:
	- sdiff key1 key2：差集
	- sinter key1 key2：交集
	- sunion key1 key2：并集
	- sdiff|sinter|sunion store deskey：将交，差，并结果保存在deskey中。
5. 使用场景：
	- SADD :标签，分类等
	- SPOP/SMEMBER：随机生成的一些场景
	- SADD+SINTER：社交功能

**5)ZSet有序集合：**
1. 使用比较少(暂时)
结构：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-17.jpg)
2. 常用API：
	- zadd key score element：添加，可批量，score可重复，element不能重复，O(logn)。
	- zrem key element：删除，可以多个，O(1)。
	- zincrby key n element：增加或减少元素的分数n,O(1)。
	- zcard：返回元素的个数,O(1)。
	- zrank：返回有序集key中成员member的排名（从小到大）
	- zrange：获取一定范围的索引的数据(按排名（从小到大）)，O((logn)+m)。
	- zrangebyscore：获取指定分数范围的数据（从小到大），O((logn)+m)。
	- zcount：返回指定范围内有多少人，O(1)。
	- zremrangebyrank:按排名范围删除元素。
4. 使用场景：
	 排行榜
5. 其他命令：
	- zrevrank：返回有序集key中成员member的排名(从大到小)，O(log(N))。
	- zrevrange，zrevrangebyscore：和上面的差不多，只是从大到小排序。
	- zinterstore：交集并存储
	- zunionstore：并集并存储
6. 总结:
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-18.jpg)

---
## 3.持久化的复习及补充
**1)持久化的作用**
1. 持久化：
redis所有数据保存在内存中，对数据的更新将会异步保存在磁盘上。
2. 持久化方式：
	- 快照：将数据拷贝保存进行备份，redis的rdb。
	- 写日志：重新执行一遍操作，redis的aof。

### RDB
**1)简介及前两种触发方式：**
1. RDB简介：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-19.jpg)
备份的是完整的数据，快照。
2. 三种生成RDB文件的方式：
	- save命令：同步命令，会产生阻塞
	- bgsave命令：异步命令
	- 自动触发：达到某些条件自动生成RDB文件
3. 三种触发方式测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-20.jpg)
save返回的是ok,而bgsave则告诉你启动了
4. save：
文件策略：存在老的RBD则用新的替换。
复杂的：O(n)
5. bgsave：
会fork一个子进程来执行bgsave操作。
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-21.jpg)
文件策略以及时间复杂度和save命令相同。
6. save与bgsave对比:
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-22.jpg)

**2）自动生成RDB文件及RDB触发机制**
1. 自动生成RDB文件。
只需要修改配置文件即可(详细查看之前的Redis学习笔记5)
补充配置：
		# bgsave出错是否暂停配置
		stop-write-on-bgsave-error yes
		# 是否使用压缩
		rdbcompression yes
		# 是否检测rdb
		rdbchecksum yes
2. 最佳配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-23.jpg)
将配置的save都删掉。(会太频繁的生成)
通常不会使用自动配置的rdb。
3. RDB触发机制：
	- 全量复制：主从复制中会使用
	- debug reload：进行debug级别的重启(不清空内存)
	- shutdown：关机

### AOF
**1)RDB的问题及AOF简介：**
1. rdb的问题:
	- 容易丢失数据：未保存时死机就会丢失数据。
	- 耗时，性能不好：将redis的所有数据dump到硬盘。
2. AOF：
将改变数据的命令记录到日志中，然后恢复时执行这些目录。

**2)AOF的三种策略：**
1. 对应了配置文件中的三种属性：
	- always：执行的每条命令都fsync到aof文件
	- everysec：每秒更新一次aof文件的命令(通常使用)
	- no：操作系统决定什么时候更新aof文件(一般不使用)
2. 三种策略的比较：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-24.jpg)

**3)AOF重写，优化：**
1. 将过期的，没有用的，重复的，抵消的命令都进行化简，以压缩aof文件。
2. 重写的目的:
	- 减少磁盘的占用量
	- 加快备份恢复的速度
3. 重写的两种实现方式：
	- bgwriteaof：类似gbsave,执行即可。
	- AOF重写的配置
4. 重写配置及统计：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-25.jpg)
触发机制：
当前尺寸>重写需要的尺寸
当前尺寸-上次尺寸/上次尺寸>AOF文件增长率
5. 重写流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-26.jpg)
6. AOF详细配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-27.jpg)

### RBD与AOF的权衡
1. RDB与AOF的比较：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-28.jpg)
2. RDB的最佳策略：
	- 减少使用(主从复制全量复制还是得使用)
	- 集中管理：集中备份
	- 主从备份时需要在从节点开一个RDB
3. AOF的最佳策略：
	- 开：缓存和存储
	- AOF重写的集中处理
	- 使用everysrc的策略
4. 通用策略：
	- 小分片：使用maxmemory来管理
	- 根据存储与缓存的特性来使用那种策略
	- 需要监控(硬盘，内存，负载，网络)
	- 准备足够的内存

---
## 4.持久化常见的问题及处理优化
**1)fork操作及改善：**
1. fork操作介绍：
	- 同步操作：会产生阻塞
	- 与内存量有关，内存越大，耗时越长(与机器类型有关)
	- info：last_fork_usec
	可查看上一次fork操作所执行的微秒数。
2. 改善fork:
	- 优先使用虚拟机或者高效支持fork的虚拟技术
	- 控制redis实例化最大使用内存：maxmemory
	- 合理配置Linux的内存分配策略：vm.overcommit_memory=1
	- 降低fork频率：例如放宽AOF重写的自动触发时机，不必要的全量复制

**2)子进程开销及优化**
1. CPU：
	- 开销：
	RDB,AOF的文件生成，属于CPU密集型
	- 优化：
	不做cpu绑定，不将cpu进程绑定到一个cpu上。
	不和cpu密集型应用部署在一起。
	单机多部署时不要出现大量的RDB生成，AOF重写等这些过程。
2. 内存：
	- 开销：
	fork内存开销，copy-on-write,子进程会共享父进程的内存快照
	- 优化：
	echo never >/sys/kernel/mm/transparent_hugepage/enabled
3. 硬盘：
	- 开销：
	AOF,RDB文件的写入，可以结合iostat,iostop等工具分析
	- 优化：
		- 不要和高硬盘负载服务部署到一起：存储服务，消息队列等
		- no-appendfsync-on-write=yes，重写期间不要执行aof的追加操作
		- 根据写入量决定磁盘类型：如SSD等
		- 单机多实例持久化文件目录可以考虑分盘

**3)AOF追加阻塞：**
1. 重写期间阻塞aof的追加操作，简单流程图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-29.jpg)
这是为了保证安全性，但是这种阻塞主线程的方式是很影响效率的，还有可能会丢失两秒的数据。
2. 问题定位：
	- 错误日志：
Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.
	- 也可以使用info persistence命令查看
	- 当然，还可以查看硬盘的状态来定位
3. 解决方案：
看上面。

---
## 5.主从复制复习及补充
**1)单机的问题及主从复制简介：**
1. 单机：
单节点，单服务器。
2. 单机的问题：
	- 机器故障
	- 容量不足
	- QPS的瓶颈(每秒查询率)
3. 分布式解决单机的各种不同问题，
而主从复制与哨兵则解决了高可用的问题。
4. 主从复制的功能(一主多从)：
	- 保证高可用，提供数据副本
	- 分流，负载均衡，读写分离
	- 扩展了Redis读的性能
5. 主从复制的规定：
	- 一个master可以有多个slave
	- 一个slave也可以有slave
	- 一个slave只能有一个master
	- 数据流向：master-->slave<-->slave
6. 简单理解：
主有啥，从也有啥。
7. 生产环境中的主从节点是不允许在一台机器上的。

**2）主从复制的两种实现方式：**
1. 两种实现方式：
	- slaveof命令操作
	- 配置实现
2. 命令实现：
在某一端口的客户端下执行slaveof就可以了，Redis会自动进行复制，但是这个复制的过程很复杂(一般是全量复制)：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-30.jpg)
如在6380端口执行：
		slaveof 127.0.0.1 6379
如果要取消主从关系，执行：
		slaveof no one
不会将6380的数据删除，而是6379新写入的值不再同步到6380
3. 配置方式：
		slaveof ip port
		slave-read-only yes
还会有其他配置查看前面的学习笔记。
4. 两种方式的简单比较：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-31.jpg)

### 全量复制及部分复制
**1)run_id及偏移量：**
1. run_id:
使用命令
`redis-cli -p 6379 -a "password" info server | grep run`
就可以查询到。
当主节点run_id发生变化时说明主节点发生了一些变化(或者从无到有)，就会将主节点的数据复制过来。
2. 偏移量
每个端口的会个字写入数据的大小(偏移量)
如果两个端口的偏移量大小相同，说明数据一致，不用复制。
如果偏移量不一致，则说明出现了主从不一致，需要使用部分复制。
查看偏移量：
`redis-cli -p 6379 -a "password" info replication | grep master_repl_offset`
注意：
主从节点的偏移量并不是完全一致的，如果有数据写入或者删除，他们都会处于一个同步更新的状态。
了解即可，一般情况下不会去关心这个值。

**2)全量复制：**
1. 希望的效果：
一个节点设置为另一个节点的从节点时，能将主节点原有的值copy过来，还能将同步复制过程中的修改也复制过来。这样就可以实现数据的完全同步。
2. 全量复制的实现示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-32.jpg)
3. 全量复制的开销：
	- bgsave的时间
	- RDB文件网络传输的时间
	- 从节点清空数据的时间
	- 从节点加载RDB的时间
	- 可能的AOF重写的时间

**3)部分复制(redis2.8之后)：**
1. 部分复制的实现原理：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-33.jpg)
2. 不管是全量复制还是部分复制都不用自己操作。

---
## 6.主从复制的问题
**1)主从故障处理：**
1. slave宕机处理：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-34.jpg)
2. master宕机处理：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-35.jpg)
3. 自动转移：高可用哨兵

### 开发运维中的问题及优化
**1)读写分离：**
1. 读写分离示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-36.jpg)
2. 可能的遇到的问题：
	- 复制数据的延迟：可以通过偏移量的监控来解决
	- 读到过期的数据：Redis3.2以后已经解决
	- 从节点发生故障：故障迁移(成本是很高的)

**2)配置不一致：**
1. 主从节点的maxmemory不一致，会进行数据淘汰，丢失数据。
使用一些标准工具安装并且对其进行监控。
2. 数据结构优化参数：内存不一致。

**3)规避全量复制：**
1. 第一次全量复制不可避免。
将第一次全量复制的消耗降低到最小：
	- 小主节点：数据分片，maxmemory的设置不要过大。
	- 低峰时进行处理：夜间，服务器压力低的时候。
2. 节点运行ID不匹配：
主节点重启，runID发生变化。
解决方案：
故障转移，哨兵或者集群。
3. 复制缓存区积压不足：
网络中断，部份复制无法完成，默认1M，偏移量在缓冲区范围内则可以进行部分复制，如果不在缓冲区范围内则需要再进行一次全量复制。
解决方案：
增大复制缓冲区，配置rel_backlog_size。
增强网络。

**4)规避复制风暴：**
1. 复制风暴：
如果一次性挂了很多个从节点，那么所有的从节点都会进行复制。Redis进行了处理，只会产生一个RDB文件，但是会进行多次传输。
2. 单主节点复制风暴：
问题：主节点重启，多从节点复制。
解决方案：更换复制拓扑。但是这种架构还是有问题，读写分离，故障转移都会有问题。其实高可用一个节点就够了。
示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-37.jpg)
3. 单机器的复制风暴：
如图所示，如果所有的主节点都在同一个机器，这台机器重启之后，那么就会出现复制灾难（大量全量复制）。
![](http://p5ki4lhmo.bkt.clouddn.com/00047Redis%E5%AD%A6%E4%B9%A06-38.jpg)
解决方案：
主从节点分散到多机器。

### 无磁盘复制
1. 通过前面的复制过程我们了解到,主库接收到SYNC的命令时会执行RDB过程,即使在配置文件中禁用RDB持久化也会生成,那么如果主库所
在的服务器磁盘IO性能较差,那么这个复制过程就会出现瓶颈,庆幸的是,Redis在2.8.18版本开始实现了无磁盘复制功能
2. 原理：Redis在与从数据库进行复制初始化时将不会将快照存储到磁盘,而是直接通过网络发送给从数据库,避免了IO性能差问题。
3. 开启无磁盘复制:repl-diskless-sync yes


---