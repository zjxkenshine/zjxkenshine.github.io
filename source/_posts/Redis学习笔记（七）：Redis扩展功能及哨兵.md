---
title: Redis学习笔记（七）：Redis扩展功能及哨兵的补充
date: 2018-4-24 9:35:49 
tags: Redis
categories: NoSQL

---
## 0.学习准备：
1. 参考资料
参考视频：慕课网教程《Redis从入门到高可用》
参考教程：<http://www.redis.net.cn/tutorial/3501.html>
Redis命令参考手册：<http://redisdoc.com/>
w3cschoolRedis教程：<https://www.w3cschool.cn/redis/nma27f21.html>
参考博客：
<https://blog.csdn.net/sunhuiliang85/article/details/74800001?utm_source=gold_browser_extension>

2. 记录一些动力节点的视频未涉及到的知识点。
包括：
	- 慢查询
	- Pipeline提高客户端效率
	- 订阅发布补充
	- Bitmap位图
	- Hyperloglog基数统计
	- Geo地理位置信息
	- 哨兵的补充

---
## 1.慢查询(日志)
主要介绍四个方面：
- 生命周期及慢查询简介
- 两个配置
- 常用命令(三个)
- 运维优化

**1)客户端请求的生命周期：**
1. 生命周期示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-01.jpg)
2. 示意图说明：
	- 出现排队的原因是Redis单线程执行命令，阻塞。
	- 慢查询是出现在第三个阶段(执行命令)：
	命令本身的执行非常慢，如keys *这种命令。
	- 客户端超时不一定慢查询，但是慢查询是客户端超时的一个可能因素。
3. 关于慢查询日志：
慢查询日志帮助开发和运维人员定位系统存在的慢操作。慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息（慢查询ID，发生时间戳，耗时，命令的详细信息）记录下来
4. 慢查询日志的组成：
慢查询日志由以下四个属性组成：标识ID，发生时间戳，命令耗时，执行命令和参数

**2)慢查询日志的两个配置：**
1. Redis如何处理慢查询日志：
	- 如果一个查询在第三阶段中被列入慢查询的范围，那么它就会被放入一个先进先出的队列--慢查询日志。默认是执行时间超过10000微秒，由slowlog-log-slower-than属性配置。注意，执行仍然是在单线程执行，慢查询日志仅用于记录和分析慢查询。
	- 慢查询日志是固定长度的，且是保存在内存中的，如果超过了这个长度，则最先进来的查询就会被踢出队列。这个长度由slowlog-max-len来记录。
2. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-02.jpg)
3. 两个配置：
	- slowlog-max-len：慢查询日志的长度，单位条。默认128。
	- slowlog-log-slower-than：慢查询日志的阈值，单位微秒，1毫秒=1000微秒。为0记录所有命令，小于0任何命令都不记录。默认值10000。
4. 可以使用config get命令得到这俩属性值：
	- config get slowlog-max-len
	- config get slowlog-log-slower-than
5. 修改方法：
	- 修改配置文件并重启(不建议)
	- 动态修改，无需重启Redis：config set
		- config set slowlog-max-len
		- config set slowlog-log-slower-than

**3)慢查询命令：**
1. 获取慢查询日志`slowlog get [n]`
n代表要获取的日志条数：
2. 获取慢查询日志列表的当前长度`slowlog len`
3. 慢查询日志重置`slowlog reset`
慢查询日志重置实际是对列表做清理操作。
4. 测试这三个命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-03.jpg)

**4)慢查询的最佳设置：**
1. slowlog-log-lower-than的设置建议
需要根据redis的并发量调整该值。由于redis采用单线程响应名利，对于高流量的场景，如果执行命令的时间在1毫秒以上，那么redis最多可支撑OPS（每秒操作次数）不到1000，因此高OPS场景的REDIS建议设置为1毫秒。

2. slowlog-max-len的设置建议
线上环境建议调大慢查询日志的列表，记录慢查询日志时Redis会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除出列表的可能性。例如线上可以设置为1000以上。

3. 理解命令的生命周期。

4. 慢查询只记录命令执行时间，并不包括命令排队时间和网络传输时间。因此客户端
命令的执行时间要大于redis服务器实际执行命令的时间。因为命令执行排队极致，慢查询会导致命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询，从而分析是否因为慢查询导致的命令级联阻塞

5. 定期持久化慢查询：
慢查询日志是一个先进先出队列，慢查询较多的情况下，可能会丢失部分慢查询命令，可以定期执行slow get命令将慢查询日志持久化到其他存储中。然后制作可视化界面查询。

---
## 2.Pipeline流水线
主要介绍：
- 什么是流水线
- 客户端实现
- 与原生操作对比
- 使用建议

**1)什么是流水线：**
1. 一次网络命令时间：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-04.jpg)
按这种情况的n次命令时间：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-05.jpg)
2. 流水线模型的n次命令时间：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-06.jpg)
像mset,mget这种命令就是流水线模型。
3. 对比：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-07.jpg)
注意两点：
	- Redis的命令执行时非常快的(微秒级别)，而且也不可避免。
	- pipeline的每次命令条数需要控制(网络环境复杂)
4. pipeline在Jedis中的实现：
未使用pipeline：
		Jedis jedis=new Jedis（"127.0.0.1",6379）;
		for(int i=0;i<10000;i++){
			jedis.hset(i,i,i);  //key,field,value
		}
这样子执行的速度会非常慢，而且主要是浪费在了网络时间上。由于key不相同所以无法使用hmset执行批量操作，所以需要使用如下流水线：
		Jedis jedis=new Jedis（"127.0.0.1",6379）;
		for(int i=0;i<100;i++){		//注意并没有将一万个操作放到一个流水线中，而是分为一百个流水线
			Pipeline pipeline=jedis.pipelined;
			for(int i=0;i<100;i++){
				jedis.hset(i,i,i);  //key,field,value
			}
			pipeline.syncAndReturnAll();
		}
这样操作可以节省大量的时间。

**2)Pipeline操作和M操作的对比：**
1. M操作的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-08.jpg)
M操作是一个原子的实现。
2. Pipeline的操作是非原子的：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-09.jpg)
但是返回的结果是按顺序的。

**3)Pipeline的使用建议：**
1. 注意每次携带的数据量。
不能无节制的往一个Pipeline中一次性添加所有的命令，注意分为多次Pipeline。
2. Pipeline每次只能在一个Redis节点上执行。
3. 注意M操作与Pipeline的区别

---
## 3.发布订阅复习及补充
主要介绍：
- 角色
- 模型
- API
- 发布订阅与消息队列的简单对比

**1)角色,模型及命令**
1. 角色：
	- 发布者publisher
	- 订阅者subscriber
	- 频道channel
2. 模型：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-10.jpg)
3. 常用命令：
	- publish channel message：发布消息到频道
	- subscribe channel...：订阅一个或者多个频道
	- unsubsc channel...：取消订阅一个或者多个频道
4. 其他命令：
	- psubscribe [partten..]：订阅符合模式匹配的所有频道
	- punsubscribe [partten..]：按照模式退订
	- pubsub channel：列出至少有一个订阅者的客户端频道
	- pubsub numsub [channel...]：列出给定一个或多个频道的订阅者总数
	- pubsub numpat：列出被订阅模式的数量

**2)消息队列和发布订阅的区别：**
1. 发布订阅是发布到某一频道后所有订阅者都会收到消息，
而消息队列则是一种抢占机制，只有一个订阅者会接收到消息。
2. 消息队列模型：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-11.jpg)

---
## 4.BitMap位图
**1)位图的简介**
1. 如字母big的位图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-12.jpg)
将每个字母转换为ASCII码的二进制表示。这就是位图。
2. 对应redis中的存储。
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-13.jpg)
可以使用getbit来读取。

**2)常用方法：**
1. `setbit key offset value`：
将key中位图表示的值的第几位(offset+1)的值设置为value，只能设置0或者1。
返回原本的值，如果之前没有值，返回0。
如：`setbit k1 2 0`
如果超过范围，则会在中间补0,如在只有20位的位图设置第51位的值：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-14.jpg)
2. `getbit key offset`：
获取指位图第offset-1位的值。
3. `bitcount key [start end]`：
获取位图指定范围中值为1的个数。不指定start和end则是全部。
4. `bitop op destkey key[key...]`：
对一个或者多个key进行交集，并集，或，异或等操作并将操作结果保存在destkey中。
5. `bitpos key targetBit [start] [end]`：
计算位图指定位数间第一个值等于targetBit的位置。
6. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-16.jpg)

**3)位图的简单使用：**
1. 使用bitMap和使用set做统计的对比(一亿用户级，50万独立用户)：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-15.jpg)
当然后面还有一个HypeLogLog结构更加节省内存。独立用户就是不重复的用户。
2. 而独立用户量很少的时候，则使用set更好。
因为BitMap需要将所有用户存入先，然后再用一个位图来将独立的用户id处置1。
3. 使用经验：
	- 是一个String类型，最大512M，如果太大则需要手动进行拆分。
	- setBit范围太大是可能耗费很多时间
	- 有时位图的效率并不一定比set好

---
## 5.HyperLogLog复习与补充
简单介绍：
- HyperLogLog结构
- 三个命令
- 内存消耗分析
- 使用经验

**1)HyperLogLog的介绍及简单使用：**
1. 详情及测试可以查看Redis学习笔记四。
2. 就是使用HyperLogLog算法，用极小的空间来进行基数统计。
3. 本质上还是String。
4. 使用方法,三个API:
	- pfadd key [element]：向HyperLogLog添加元素。
	- pfcount key [key]：计算HyperLogLoge的独立基数。
	- pfmerge destkey key[key...]：对多个key(HyperLogLog)进行合并。

**2)内存消耗分析及使用经验：**
1. 内存消耗分析(百万级用户基数)：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-17.jpg)
至于为什么一天15KB，有些博客说是12KB，如下：
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。以后啃算法的时候再去深究吧。
2. 使用经验(HyperLogLog的注意事项)：
是否能够容忍错误：官方给出的错误率是0.81%
HyperLogLog无法取出条数据。

---
## 6.GEO地理位置信息
**1)GEO的简介：**
1. GEO(地理信息定位)：存储经纬度，用于计算两地距离，范围计算等。

**2)常用命令：**
1. GEOADD：
`GEOADD key longitude latitude member [longitude latitude member ...]`
	- 将给定的空间元素（纬度、经度、名字）添加到指定的键里面。
这些数据会以有序集合的形式被储存在键里面， 从而使得像 GEORADIUS 和 GEORADIUSBYMEMBER 这样的命令可以在之后通过位置查询取得这些元素。
	- GEOADD 命令以标准的 x,y 格式接受参数， 所以用户必须先输入经度， 然后再输入纬度。 GEOADD 能够记录的坐标是有限的： 非常接近两极的区域是无法被索引的。
	- GEOADD的经纬度是有范围的，超过这个范围将会报错：
		- 有效的经度介于 -180 度至 180 度之间。
		- 有效的纬度介于 -85.05112878 度至 85.05112878 度之间。
2. GEOPOS：
`GEOPOS key member [member ...]`
从键里面返回所有给定位置元素（一个或多个）的位置（经度和纬度）。
3. GEODIST：
`GEODIST key member1 member2 [unit]`
计算两个位置之间的距离，默认以米做单位。
指定单位的参数 unit 必须是以下单位的其中一个：m 米，km 千米，mi 英里，ft 英尺。
4. GEORADUIS:计算指定范围内的集合，中心点是经纬度决定的
GEORADUISBYMEMBER：计算指定范围内的集合，中心点是member决定的
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-18.jpg)
5. GEOHASH:
`GEOHASH key member [member ...]`
返回一个或多个位置元素的 Geohash 表示。
6. 删除则使用`ZREM key member`。
7. 简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-19.jpg)

---
## 7.哨兵Sentinel的复习及补充
简要目录：
- Redis主从复制的问题
- Sentinel架构说明
- Sentinel安装配置
- 客户端连接Sentinel
- Sentinel实现原理
- 常见开发运维问题

**1)主从复制缺陷及解决：**
1. 需要手动处理故障：
使用哨兵Sentinel来解决
2. 写能力和存储能力受限：只能在一个节点上进行写和存储。
使用分布式，集群来解决。
3. Redis Sentinel的基本架构：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-20.jpg)
Sentinel是特殊的Redis
4. Sentinel的故障转移(自动)：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-21-1.jpg)
可以自动实现监控，故障转移及回报客户端。
5. Sentinel是可以实现对多套主从的监控：使用master-name进行配置
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-22.jpg)

**2)Redis Sentinel的安装及配置使用：**
1. 安装其实就是在Sentinel中声明监控的master-name
2. 具体的安装配置使用过程查看前面的Redis学习笔记(五)
3. 主配置文件配置完毕后可以使用如下方法来赋值从节点的配置文件：
		# sed "s/6379/6380/g" redis-6379.conf > redis6380.conf
		# sed "s/6379/6381/g" redis-6379.conf > redis6381.conf
这样直接就赋值并且修改好了。
4. 其余的Sentinel的文件的配置启动都和前面相同。
5. 对前面使用的补充：
客户端可以直接连到sentinel上(也应该这样连接，前面可能只是实验)：
		# redis-sentinel -p 26379
这种写法，并且能进行客户端操作。
使用info sentinel会显示一些该sentinel的信息,各个sentinel是可以互相感知的。

**3)Java客户端如何连接到Redis Sentinel：**
1. Redis Sentinel的客户端请求流程：
第一步：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-23.jpg)
第二步：客户端从Sentinel获取master节点
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-24.jpg)
第三步：客户端验证是否是master节点
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-25.jpg)
第四步：master发生变化通知并得到一个新的master
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-26.jpg)
通过订阅该master节点的频道实现。
2. 客户端的基本流程框架：和Sentinel的架构是一样的
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-20.jpg)
3. 客户端接通Sentiel的流程：
	- 需要一个Sentiel地址的集合，选择一个能ping同的节点然后连接。
	- 需要使用masterName配置的值
	- 它不是一个代理模式
4. 如何使用Jedis连接一个Sentinel（有些知识poolconfig等后面补充）：
		JedisSentinelPool jedisSentinelPool =new JedisSentinelPool(masterName, sentinels, 
				poolConfig, connectionTimeout, soTimeout, password, database, clientName);
这是使用Sentinel池最多参数的一个构造方法，一般只用前两个参数即可。
5. 网上的简单连接例子：
		Set<String> IPS = new HashMap<String>();  
		IPS.add("192.168.0.1:26379");  
		JedisSentinelPool pool = new JedisSentinelPool("mymaster", IPS);  
		
		public synchronized String set(String key, String value) throws Exception{  
			Jedis jedis = ;  
			try {  
				String hostIp = pool.getCurrentHostMaster().getHost();  
				int hostPort = pool.getCurrentHostMaster().getPort();  
				WifiLogUtil.wifiLogInfo("Jedis set", "Server >> ["+hostIp+":"+hostPort+"]");  
				jedis =new Jedis(hostIp, hostPort);  
			if(value == )  
					return jedis.set(key, value);  
				return jedis.set(key, new String(value.getBytes(), "UTF-8"));  
			} catch (Exception e) {  
				WifiLogUtil.wifiLogError(e, "jedis set", "set jedis error : "+e);  
				throw e;  
			} finally {  
				if(jedis != )
					//池的环境解释为归还
				jedis.close();  
			}
		}
故障转移时需要一段时间，之后就可以了。
6. 后面如果涉及到分片的话就得自己实现ShardedJedisSentinelPool了，这都是后话了。

### Sentinel的实现原理
**1)三个定时任务：**
Sentinel对redis节点做失败判定和故障转移是通过三个定时任务来实现的：
1. 第一个定时任务：每10秒每个sentinel对master和slave执行info：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-27.jpg)
主要作用：
	- 发现slave节点
	- 确认主从关系
2. 第二个定时任务：每2秒每个sentinel通过master节点的channel交换信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-28.jpg)
主要作用：
	- 通过_sentinel_：hello频道交互。
	- 交互对节点的看法和自身信息。
	- (心跳检测，可以达成一个信息的交互感知)
3. 第三个定时任务：每秒每个sentinel对其他sentinel和所有Redis执行一次PING。
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-29.jpg)
主要作用：
	- 检测故障，故障转移中最重要的一步。

**2)主观下线及客观下线：**
1. 在配置文件中有个这个选项：
`sentinel monitor <masterName> <ip> <masterPort> <法定人数>`
如有三个sentinel，可以配置如下：
		sentinel monitor mymaster 127.0.0.1 6380 2  //超过两个人则自动故障处理
2. `sentinel down-after-milliseconds <masterName> <timeout>`
master或slave多长时间（默认30秒）不能使用后标记为down状态
3. 主观判断与客观判断：
主观判断(下线)：每个Sentinel对Redis节点下线的判断。
客观判断(下线)：所有Sentinel对节点下线达成的共识(大于等于法定人数)
所以Sentinel的数量最好是3个以上的奇数。
4. 如果有master宕机了，感知到这个master下线的Sentinel会向其他Sentinel发送一个`sentinel is-master-down-by-address`，其他Sentinel会进行投票，通过后进行故障转移。
5. slave节点下线不会有这些机制，因为Sentinel只监视master。

**3)领导者选举：**
1. 进行投票的原因就是领导者选举。
2. 而领导者选举的原因是执行故障转移只需要一个领导者。
3. 选举：通过`sentinel is-master-down-by-address`命令说明希望成为领导者。
4. 选举的规则：
	- 每个做主观下线的Sentinel节点向其他Sentinel发送命令，要求成为领导者。
	- 收到命令的节点如果没有同意通过其他节点的请求，那么会同意该请求，否则拒绝。
	- 当一个Sentinel发现自身的票数超过法定人数时，自动成为领导者，并进行故障处理。
	- 如果此过程中有多个节点成为领导者，那么将等待一段时间之后进行重新选举。
5. 领导者选举使用的是一个Ruby的算法。

**4)故障转移的实现，过程：**
1. 简单的过程：
	- 1.从slave节点中选择一个“合适的”节点作为新的master节点
	- 2.对选择的slave节点执行`slaveof no one`使其成为master节点
	- 3.向剩余的slave节点发送命令，让他们成为新的master节点的slave节点。
	复制规则和parallel-syncs参数有关。
	- 4.将原来的master节点设置为slave,并保持对其的关注,这个节点恢复开启之后让它去复制新的master节点。
2. 合适的节点：
	- 1.选择slave-priority(slave节点优先级)最高的节点，存在则返回，不存在则继续。(一般不配置)
	- 2.选择数据偏移量更大的节点(复制的最完整)，有则返回，没有则继续。
	- 3.选择run_id最小的节点，启动最早的。

### 常见的运维方式
主要介绍两点：
- 节点运维
- 高可用分离

**1)节点运维：**
1. 主要的任务：节点上线和下线
2. 主要针对目标：Sentinel节点，主节点，从节点
3. 节点运维的原因：
	- 机器下线：例如过保的情况
	- 机器性能不足：如CPU、内存、硬盘、网络等
	- 节点自身故障：例如服务不稳定等
4. 主节点下线操作：
	- 1.执行命令：`sentinel failover <mastername>`
	- 2.然后执行故障转移，不会执行其他过程如选举等。
	- 然后进行从节点下线
5. 主节点执行`sentinel failover <mastername>`后内部运行过程：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-30.jpg)
6. 从节点和Sentinel节点下线主要考虑的情况：
要确定是暂时下线还是永久下线，永久下线则要做一些清理工作，一些配置，日志，数据(AOF,RDB)等进行清理。
需要考虑读写分离的情况。
7. 节点上线：
	- 主节点：`sentinel failover`命令进行替换
	- 从节点：`slaveof no one`可升为主节点，sentinel可以感知
	- sentinel节点：参考其他sentinel启动即可。

**2)高可用读写分离：**
1. 先看一看JedisSentinelPool的实现:
		public JedisSentinelPool(String masterName, Set<String> sentinels,final GenericObjectPoolConfig poolConfig, final int connectionTimeout, final int soTimeout,final String password, final int database, final String clientName) {
				this.poolConfig = poolConfig;	//池配置
				this.connectionTimeout = connectionTimeout;	//连接超时
				this.soTimeout = soTimeout;		//读写超时
				this.password = password;	//密码
				this.database = database;	//数据库
				this.clientName = clientName;	//客户端名字
		
				HostAndPort master = initSentinels(sentinels, masterName);	//初始化
				initPool(master);	//连接池的初始化
		}
2. initSentinels的源码：客户端实现的原理
		private HostAndPort initSentinels(Set<String> sentinels, final String masterName) {
			HostAndPort master = null; //ip和端口
			boolean sentinelAvailable = false;	//sentinal可用性
			log.info("Trying to find master from available Sentinels...");
			//遍历所有的sentinel节点(主要是查看他们监控的master)
			for (String sentinel : sentinels) {
				final HostAndPort hap = HostAndPort.parseString(sentinel); //解析得到HostAndPort对象
				log.fine("Connecting to Sentinel " + hap);
		
				Jedis jedis = null;
				try {
					jedis = new Jedis(hap.getHost(), hap.getPort());	//连接这个ip及host
					List<String> masterAddr = jedis.sentinelGetMasterAddrByName(masterName);  //得到master地址
					sentinelAvailable = true;
		
					if (masterAddr == null || masterAddr.size() != 2) {
						log.warning("Can not get master addr, master name: " + masterName + ". Sentinel: " + hap + ".");
						continue;	//地址空说明遍历失败，遍历下一个节点
					}
		
					master = toHostAndPort(masterAddr);	//通过该方法得到master
					log.fine("Found Redis master at " + master);
					break;
				} catch (JedisException e) {
					log.warning("Cannot get master address from sentinel running @ " + hap + ". Reason: " + e + ". Trying next one."); //警告信息
				} finally {
					if (jedis != null) {
						jedis.close();
					}
				}
			}
		
			if (master == null) {
				if (sentinelAvailable) {
					throw new JedisException("Can connect to sentinel, but " + masterName + " seems to be not monitored...");	//如果遍历所有节点都没有master则报异常
				} else {
					throw new JedisConnectionException("All sentinels down, cannot determine where is " + masterName + " master is running...");
				}
			}
		
			log.info("Redis master running at " + master + ", starting Sentinel listeners...");
			
			for (String sentinel : sentinels) {	
				final HostAndPort hap = HostAndPort.parseString(sentinel);
				MasterListener masterListener = new MasterListener(masterName, hap.getHost(), hap.getPort());  //MasterListerner是一个订阅线程，订阅了一个switch-master 的频道
				masterListener.setDaemon(true);
				masterListeners.add(masterListener);
				masterListener.start();
			}
			return master;
		}
3. 从节点的作用：
 	- 副本
 	- 高可用的基础
4. 关心slave节点主要是三个消息：需要自定义客户端来实现
	- +switch-master：切换主从节点(从节点晋升主节点)。
	- +convert-to-slave：主节点切换到从节点
	- +sdown：主观下线
5. 简单图示：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-31.jpg)
具体的还是得自己实现。使它能检测那几个消息并且能进行相应的处理。(读客户端变成写客户端，写客户端变成读客户端)
6. 但是这样比较复杂，真正需要扩展的时候建议使用集群。

---
## 8.客户端连接的补充操作--Jedis连接池
**1)Jedis简介：**
1. 获取Jedis:Maven依赖
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
			<version>2.9.0</version>
			<type>tar</type>
			<scope>complier</scope>
		</dependency>
2. Jedis直连：
生成一个Jedis对象，这个对象负责和指定的节点进行通信。(soket实现）
		Jedis jedis=new Jedis("127.0.0.1","6379");
jedis执行set操作
		jedis.set("k1","v1");
执行get操作：
		jedis.get("k1"）;
关闭Jedis连接：
		jedis.close;
3. 方法介绍：
常用的构造方法是Jedis(host,port,connectionTime,soTime):
	- host：ip
	- port：端口
	- connectionTime：客户端连接超时
	- soTime：客户端读写超时

**2)Jedis连接池的使用：**
1. 直连的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-32.jpg)
2. Jedis连接池示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-33.jpg)
3. 直连与池的对比：
![](http://p5ki4lhmo.bkt.clouddn.com/00048Redis%E5%AD%A6%E4%B9%A07-34.jpg)
4. 关于Jedis连接池的实现：
		public Jedis getJedisBypool(){
			GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
			//连接Jedis服务器
			JedisPool jedpool =new JedisPool(poolConfig,"10.186.151.205", 6379);
			Jedis jed=null;
			try{
				jed=jedpool.getResource();
				//输入密码，否则只能登陆而不能获取数据
				jed.auth("123456");
			}catch(Exception e){
				e.printStackTrace();
			}
			return jed;
		}
5. 这个GenericObjectPoolConfig是org.apache.commons.pool2类中的一个子类。
其中的配置如下：
		//默认最大连接数
		public static final int DEFAULT_MAX_TOTAL = 8;
		//最大空闲连接数
		public static final int DEFAULT_MAX_IDLE = 8;
		//最小空闲连接数
		public static final int DEFAULT_MIN_IDLE = 0;
		
		private int maxTotal = DEFAULT_MAX_TOTAL;
		private int maxIdle = DEFAULT_MAX_IDLE;
		private int minIdle = DEFAULT_MIN_IDLE;

---

