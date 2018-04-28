---
title: Redis学习笔记（九）：集群伸缩与常见运维问题
date: 2018-4-26 17:49:40  
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
	- 集群伸缩
		- 伸缩原理
		- 扩容集群
		- 集群缩容
	- 客户端路由(JedisCluster)
		- moved重定向
		- ask重定向
		- smart客户端(JedisCluster)
			- 实现原理
			- 使用
			- 多节点命令操作
			- 批量操作
	- 故障转移原理
		- 故障发现
		- 故障恢复
	- 开发运维常见问题

---
## 1.伸缩原理
1. 集群伸缩的简单原理图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-01.jpg)
2. 集群扩展的简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-02.jpg)
添加节点后原有的节点会移动一些槽到新有的节点上。实际上移动的就是键值对。
3. 集群伸缩的原理：
槽，数据在节点之间的移动

---
## 2.集群扩容
简单来说就是增加节点
**1)简单流程：**
1. 简单分为三步：	
	- 准备新节点(一对)
	- 加入集群
	- 分配槽和数据
		- 设置槽迁移计划
		- 迁移数据
		- 设置主从
2. 第一步准备节点比较简单：
配置
`# sed 's/8000/8006/g' redis-8000.conf > redis-8006.conf`
开启：
`# redis-server redis-8006.conf`
`# redis-server redis-8007.conf`
3. 此时这俩节点是这样的：(视频的图)
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-03.jpg)
并未加入到这个集群中。他们被称为孤立节点。

**2)将孤立节点加入集群：**
1. 加入集群的作用：
	- 为它迁移槽和数据实现扩容
	- 作为从节点负责故障转移(某从节点需要下线)
2. 原生命令方式：在某一集群中客户端中执行
		cluster meet 127.0.0.1 8006
		cluster meet 127.0.0.1 8007
查看加入后的节点状态：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-04.jpg)
但是这种方式有一个问题：
如果新节点已经加入了某个集群，则会造成故障。
新加入的两个节点需要额外执行slaveof来设置主从关系。
--》所以建议是有后面Ruby工具的方式
3. 使用工具添加节点：
		# redis-trib.rb add-node 新节点ip:新节点端口 已存在节点ip:已存在节点端口 
这两个节点的操作是使他们做一个meet操作。还可以加参数，如下：
		# redis-trib.rb add-node 127.0.0.1：8006 127.0.0.1：8000 --slave --master-id <arg>
add-node有两个可选参数：
--slave：设置该参数，则新节点以slave的角色加入集群
--master-id：这个参数需要设置了--slave才能生效，--master-id用来指定新节点的master节点。如果不设置该参数，则会随机为节点选择master节点。

### 迁移槽和数据
这是集群扩容或者缩容的主要步骤，主要分为三步:
- 槽迁移计划
- 迁移数据
- 添加从节点

**1)槽迁移计划：**
1. 肯定要平分槽数量：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-05.jpg)
2. 理解迁移的大致过程：每个槽拿出约1366个节点给新的槽
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-06.jpg)

**2)迁移数据：以下是完整迁移一个槽的过程**
1. 对目标节点发送命令使其准备导入槽的数据：
		cluster setslot {slot} importing {sourceNodeId} //源节点的Id
2. 对源节点发送命令使其准备导出槽的数据：
		cluster setslot {slot} migrating {targetNodeId} //也是使用
3. 对这个槽进行循环遍历，每次获取一定数量的keys:
		cluster getkeysinslot {slot} {count}
每次获取conut个属于槽的键。
4. 在源节点执行migrate操作迁移指定的key:
		migrate {targetIp} {targerPort} key 0 {timeout}  //0是指数据库，后面是延迟时间
MIGRATE 命令需要在给定的时间规定内完成 IO 操作。如果在传送数据时发生 IO 错误，或者达到了超时时间，那么命令会停止执行
5. 重复执行第三步和第四步，直到所有的键都迁移完成。
6. 向集群内所有主节点发送：
		cluster setslot {slot} node {targetNodeId}
通知将槽分配给目标节点了。(通知所有主节点该槽归目标节点管理了)
7. 以上6步只是迁移了一个槽，如果要迁移一千多个节点还需要执行一千多次1~6的过程。这时候就必须要用shell脚本了。
8. 简单流程图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-07.jpg)
9. Redis3.0.6开始提供了一个Pipeline migrate用于批量迁移key：(第四步)
但是这种方式会丢失数据。(3.2.8已经修复)
10. 网上好多介绍Redis-Migrate-Tool工具的，有空再研究一下。

**3)使用Redis-trib.rb的Reshard方法来快捷迁移：**
1. 在Linunx命令行中输入redis-trib.rb可以看到以下的方法介绍：
		reshard		host:port		//ip：端口，可任意指定，只是为了执行reshard
						--from <arg>
						--to <arg>
						--slots <arg>
						--yes
						--timeout <arg>
						--pipeline <arg>

2. 其实只要简单的使用`redis-trib.rb reshard host:port`就可以了，会出现很多提示，按照提示进行操作就可以。
3. 测试：
使用命令`redis-trib.rb reshard 127.0.0.1:8000`,它会打印出各项集群中的信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-08.jpg)
然后根据信息输入要迁移多少个槽，迁移到哪个ID以及从哪个节点迁移：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-09.jpg)
all则表示从所有节点分配，如果输错了可以使用ctrl+u来删除。
之后会有一个选择，执行就好，然后就会开始移动槽：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-10.jpg)
等到移动结束就可以了。
4. 这其中会出现很多的问题，具体见Redis的问题总结，有几个有用的方法在这里总结一下：
		redis-trib.rb rebalance ip:port		//使槽点分配均匀
		redis-trib.rb fix ip:port		//修复某些错误
		redis-trib.rb check ip:port		//查看该集群槽点的分配情况，检测槽点
		redis-trib.rb info ip:port		//查看节点，槽，键值的分布

**4)设置主从：**
1. 如果前面使用的是meet进行节点的访问，那么这时就需要设置主从了。
和前面构建好集群时一样，在希望从的节点下执行：
		cluster replicate node-id
2. 最后使用`redis-trib.rb check ip:port`查看扩容后的集群：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-11-1.jpg)

---
## 3.集群收缩
**1)集群收缩的简单过程：**
1. 主要分为三步：
	- 下线，迁移槽和数据
	- 忘记节点
	- 关闭节点
2. 简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-12.jpg)
3. 下线迁移槽的过程：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-13.jpg)
4. 迁移槽和数据的方法和前面扩容时迁移槽和数据的方法一模一样。
不再过多介绍了。

**2)忘记节点与关闭节点：**
1. 忘记节点(下线节点)：
		cluster forget {targetNodeId}
如想要让8000节点忘记8006节点：在8000客户端下执行：
		cluster forget 8006节点的Id
2. 如果要使它离开集群，则需要在所有的节点都执行cluster forget：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-14.jpg)
这个60秒有效是指60秒后如果集群中依然有节点和改节点产生未断开连接，则该节点会重新和那个节点产生通信。
3. 关闭节点，就是关闭Redis服务，kill掉就可以了。
4. `redis-trib.rb`提供的忘记节点的方法：
		# redis-trib.rb del-node host:port node-id
host:port可以随意指定，只是指定一个执行该方法的端口。
下线节点时记得先下线从节点再下线主节点。

---
## 4.moved重定向与ask重定向
**1)moved重定向：**
1. 简单示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-15.jpg)
注意第三步传回客户端后还需要自己编写代码去执行第四步。
2. 槽命中的情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-16.jpg)
3. 槽未命中的情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-17.jpg)
moved异常，但是在登录客户端时使用-c参数就会忽略这个异常。
-c的意思就是使用集群模式，自动完成跳转(捕获move异常，并跳转到目标节点)。
4. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-18.jpg)
如果没有使用-c参数，则会报异常，而且需要手动跳转。
5. Jedis的客户端JedisCluster的实现原理和这个是相同的

**2)ask重定向：**
1. 产生的背景：
集群扩容缩容时需要迁移槽，而槽的迁移需要花费很长的时间，所以无法很精确的判断slot的位置。可能原来是在源节点，访问时就变成目标节点了。
如下图所示：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-19.jpg)
2. 实现的过程：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-20.jpg)
3. 会报一个ask专向异常，客户端也会自动为我们处理。

**3)moved重定向和ask重定向的异同：**
1. 相同点：
	- 都是在客户端进行重定向
	- 但是Redis cluster都已经给我们处理好了
2. 不同点：
	- moved重定向的槽是确认迁移完毕了的
	- ask重定向的槽是还在迁移中的
3. 给实现客户端带来的挑战：
	- 节点迁移
	- 性能问题(高命中)

---
## 5.smart客户端(JedisCluster)
**1)客户端原理：**
1. 客户端原理：
	- 1.从集群中选择一个可运行的节点，使用cluster slots初始化槽和节点映射。
	- 2.将cluster slots结果映射到本地，为每个节点创建JedisPool
	- 3.准备执行命令。
2. 在执行命令之前需要明白的一点是，官方推荐的是使用直接连接的方式，虽然可以使用代理来处理连接到哪个节点，但是会有性能问题，所以尽量连接到key对应的slot所在的节点上。当然对Moved异常和ask异常也会提供必要的支持。
3. 执行命令的基本流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-21.jpg)

**2)JedisClusterCommand类源码：实现方法**
1. 源码：上述执行命令的处理逻辑
		private T runWithRetries(byte[] key, int attempts, boolean tryRandomNode, boolean asking) {
			if (attempts <= 0) {
				throw new JedisClusterMaxRedirectionsException("Too many Cluster redirections?");
			}
		
			Jedis connection = null;
			try {
				if (asking) {	//判断是否是asking的状态
					connection = askConnection.get();
					connection.asking();
					asking = false;
				} else {
					if (tryRandomNode) {	//是否在随机节点执行
						connection = connectionHandler.getConnection();		//得到对应的连接
					} else {
						connection = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.getSlot(key));	//算出对应的连接(节点)并返回
					}
				}
			return execute(connection);	//执行命令
			} catch (JedisNoReachableClusterNodeException jnrcne) {
				throw jnrcne;	//无节点可达，直接抛出异常
			} catch (JedisConnectionException jce) {	//连接异常，不属于槽和节点连接变化的那种(moved,asking)异常
				releaseConnection(connection);	//释放原来的连接
				connection = null;
				if (attempts <= 1) {
					this.connectionHandler.renewSlotCache();	//刷本地缓存
					throw jce;
				}
				return runWithRetries(key, attempts - 1, tryRandomNode, asking);
			} catch (JedisRedirectionException jre) { //重定向异常
				if (jre instanceof JedisMovedDataException) { //moved异常
					this.connectionHandler.renewSlotCache(connection);	//刷新本地缓存
				}
				releaseConnection(connection);
				connection = null;
				if (jre instanceof JedisAskDataException) { //ask异常
					asking = true;	//设置asking为true
					askConnection.set(this.connectionHandler.getConnectionFromNode(jre.getTargetNode()));	//重新连接
				} else if (jre instanceof JedisMovedDataException) {
				} else {
					throw new JedisClusterException(jre);
				}
				return runWithRetries(key, attempts - 1, false, asking);
			} finally {
				releaseConnection(connection);
			}
		}
2. 不要轻易刷新本地缓存，只有在出现moved异常的时候再刷新，提高效率。

### JedisCluster的使用
1. 主要介绍以下三点：
	- RedisCluster基本用法
	- 多节点命令的实现
	- 批量命令的实现
2. 视频里还有介绍整合Spring的，以后再说吧。

**1)基本使用：**
1. 简单的使用代码：
		public JedisCluster getJedisCluster(){
			//定义一个节点连接集合
			Set<HostAndPort> shap=new HashSet<HostAndPort>();
			shap.add(new HostAndPort("127.0.0.1", 8000));
			shap.add(new HostAndPort("127.0.0.1", 8001));
			shap.add(new HostAndPort("127.0.0.1", 8002));
			shap.add(new HostAndPort("127.0.0.1", 8003));
			shap.add(new HostAndPort("127.0.0.1", 8004));
			shap.add(new HostAndPort("127.0.0.1", 8005));
			shap.add(new HostAndPort("127.0.0.1", 8006));
			shap.add(new HostAndPort("127.0.0.1", 8007));
			GenericObjectPoolConfig poolConfig =new GenericObjectPoolConfig();
			JedisCluster jclu=new JedisCluster(shap,1000,poolConfig);
			return jclu;
		}
2. 通过这个方法得到JedisCluster连接之后就可以了执行各种命令了。
不需要什么归还等的那种工作，JedisCluster默认封装了这些工作。
3. 使用技巧：
	- 单例：内置了所有节点的连接池
	- 无需手动借还连接池，但是仍然需要close销毁方法
	- 合理设置Commons-pool的配置
	- 注意JedisPool以及JedisSentinelPool的干扰

**2)多节点命令的操作：**
1. 简单来说，就是能在所有节点上执行命令。
2. 简单的代码如下：
		//多节点操作
		public void multiNodes(){
			Map<String,JedisPool> jedisPoolMap=getJedisCluster().getClusterNodes();
			for(Entry<String, JedisPool> entry:jedisPoolMap.entrySet()){
				//获取每个节点的Jedis连接
				Jedis jedis=entry.getValue().getResource();
				//判断是不是主节点(代码未知)
				//执行命令
				jedis.close();
			}
		}

**3）批量操作怎么实现：**
1. 如hmset,hmget这些批量操作(hmset那些不算)，如何将批量操作的key都放到一个槽里呢?（Redis的要求）
2. 串行mget解决方案：能解决但是效率差(n次网络时间)
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-22.jpg)
3. 串行IO解决方案：节省网络时间(只需要node次网络时间)
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-23.jpg)
4. 并行IO：只需要1次网络时间，并行操作
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-24.jpg)
5. hash_tag方式：一次网络时间，且所有key都落在一个节点
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-25.jpg)
6. 四种方式的优缺点：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-26.jpg)

---
## 6.故障转移
主要介绍：
- 故障发现
- 故障恢复

**1)故障发现：**
1. Redis集群中并没有提供Sentinel这种监控，但是仍然可以实现高可用，因为集群是通过其他节点进行监控的。
2. 通过节点之间的ping/pong消息，不需要依赖Sentinel。
3. 分为主观下线和客观下线。
4. 主观下线：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-27.jpg)
5. 客观下线：半数以上持有槽的主节点都标记某节点下线
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-28.jpg)
尝试客观下线的流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-29.jpg)

**2)故障恢复：**
1. 主要分为四步：
	- 资格检查
	- 准备选举时间
	- 选举投票
	- 替换主节点
2. 资格检查：
检查哪些从节点有替换该主节点的资格。
	- 每个从节点检查与故障主节点的断线时间
	- 超过cluster-node-timeout\*cluster-slave-validity-factory则取消资格。(默认是15\*10=150秒)
3. 准备选举时间：
为了保证偏移量最大的从节点有更小的延迟达到选举时间。(为了保证数据的一致性)
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-30.jpg)
4. 发起选举投票：
由主节点进行投票，偏移量越大的从节点优先级越高。
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-31.jpg)
5. 替换主节点：
	- 当前从节点取消复制变为主节点(slaveof no one)
	- 执行ClusterDelSlot撤销故障主节点负责的槽，并执行ClusterAddSlot把这些槽分配给自己。
	- 向集群广播pong消息，表明自己已经替换故障的主节点。

---
## 7.集群开发维护问题
目录：
- 集群完整性
- 带宽消耗
- Pub/Sub广播
- 数据倾斜
- 读写分离(集群下的读写分离)
- 数据迁移(单机迁移到集群)
- 集群和单机的对比

### 完整性，带宽消耗以及Pub/Sub广播
**1)集群完整性：**
1. cluster-require-full-coverage默认为yes
	- 集群中所有的16384个槽都必须可用才能提供服务：保证集群完整性
	- 此时节点故障或者正在转移故障时,访问任意节点，就会报错：
	(error)CLUSTERDOWN The cluster is down
2. 但是绝大多数的业务都是难以容忍的，所以一般配置成no：
		cluster-require-full-coverage=no

**2)带宽消耗：**
1. RedisCluster节点之间的信息交换，心跳检测示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-32.jpg)
2. 官方建议最好不要超过1000个节点。
节点过多进行ping/pong消息交换时会产生大量的带宽消耗
3. 带宽消耗的说明：三个方面
	- **消息发送频率**：节点发现它与其他节点的最后通信时间超过cluster-node-timeout/2时就会直接发送Ping消息
	- **消息数据量**：slots槽数组(2k)和整个集群的状态数据的十分之一(10个节点状态数据约为1k)
	- **节点部署的机器规模**：集群内分布的机器数量越多且每台机器划分的节点数量约均匀则可用的带宽越高。
4. 相关优化：
	- 避免“大”集群：避免多业务使用同一个集群，大业务可以使用多个集群。
	- cluster-node-timeout即影响带宽也影响故障转移的时间，需要在这两者之间进行均衡(越大则故障转移时间越长，产生带宽消耗越小)
	- 尽量将节点均匀分布到尽量多的机器上，可以保证高可用和带宽。

**3)Pub/Sub广播在集群环境下产生的问题：**
1. 集群下使用Pub/Sub广播时，消息会在各个节点之间传播，会严重消耗带宽（对每个节点进行广播）。
示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-33.jpg)
2. 解决方案：
单独建立一套Sentinel的服务来处理Pub/Sub消息。(Pub/Sub不适合集群)

### 集群倾斜
主要介绍两点：
	- 数据倾斜：内存不均(某些节点内的数据非常多)
	- 请求倾斜：某个key请求量非常高

**1)数据倾斜：**
1. 简单的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00051Redis%E5%AD%A6%E4%B9%A09-34.jpg)
2. 引起数据的倾斜的原因：
	- 节点和槽的分配不均匀
	- 不同槽对应的键值数比较大
	- 包含bigkey(一个key中的数据非常多)
	- 内存相关的配置不一致(不常见)
	- 客户端缓冲区(不常见)
	- 哈希表扩容异常(不常见)
3. 节点和槽分配不均匀：
查看节点，槽，键值对的分布：
`redis-trib.rb info ip:port`
均衡节点和槽(谨慎使用)：涉及到槽操作最好使用客户端
`redis-trib.rb rebalance ip:port`
对键的迁移时最好自己制定迁移计划而不是使用rebalance。
4. 不同槽对应的键值数比较大：
	- CRC16算法在一般情况下比较均匀
	- 可能存在hash_tag
	- 可以使用如下命令获取对应的key的数量：
	`cluster countkeysinslot {slot}`
	- 获取槽中某些key:number是要获取多少个key
	`cluster keysinslot {slot} number`
5. 包含bigkey：
	- 大的字符串，几百万元素的hash,list,set等
	- 从节点：`redis-cli --bigkeys`找到bigkey
	- 优化：
	优化数据结构(拆分成很小的key)
6. 内存相关的配置不一致：
	- hash-max-ziplist-value,set-max-intset-entris等
	如果其他节点都配置了，有一个节点未配置，则会导致数据的倾斜
	- 优化：
	定期检查集群的一致性

**2)请求倾斜：**
1. 热点key：重要的key或者BigKey
重要的key：比如某个明星的消息成为热点保存在某一key中
BigKey：在很多业务上会使用
2. 优化：三方面
	- 避免BigKey
	- 热键不要用hash_tag
	- 一致性不高时，建议使用本地缓存+MQ

### 读写分离，数据迁移与和单机比较
**1)读写分离：**
1. 只读连接：
在集群模式下从节点不接受任何读写请求(读也默认不接受)
	- 默认会重定向到负责该槽的主节点
	- 有一个连接级别的命令readonly可以实现只读：
			redis-cli -c -p 8004 -a 123456
			127.0.0.1:8004>readonly
			...
	每次连接都需要执行一个readonly
2. 读写分离：比单机更加复杂
和单机有同样的问题：详见Redis学习笔记(七)
	- 复制延迟
	- 读取过期数据
	- 从节点故障
3. 但是不建议使用读写分离：
读写分离需要实现自己的客户端。
有一个命令`cluster slaves {nodeId}`来获取对应的从节点
还需要考虑节点和槽的关系，读写分离的成本非常高。
4. 建议扩大集群规模，而不要轻易选择读写分离。

**2)数据迁移(离线/在线)**
1. 数据迁移就是将单机的数据迁移到集群中或者反过来。
2. 官方迁移工具：`redis-trib.rb import`
	- 只支持从单机迁移到集群，不支持集群到单机。
	- 不支持在线迁移，source需要停写
	- 不支持断点续传
	- 单线程迁移，影响速度
3. 在线迁移工具(比较好用的)：支持在线迁移
	- 唯品会：`redis-migrate-tool`
	- 豌豆荚：`redis-port`

**3)集群与单机的比较：**
1. 集群的限制：
	- key批量操作支持有限：mget,mset必须保证在同一个槽(slot)
	- key事务和Lua的支持有限：操作的key必须在一个节点上
	- key是数据分区的最小粒度：不支持BigKey分区
	- 不支持多个数据库：在集群模式下只有一个数据库db0
	单机状态下也不建议使用多数据库
	- 复制只支持一层：不支持树型的复制(slave复制slave)
2. Redis Cluster：满足容量和性能的扩展性，但是很多业务的QPS达不到这种需求
	- 对于大多数客户端来说性能会降低
	- 无法跨节点使用一些命令：mget,keys,scan,flush,sinter等
	- Lua和事务无法跨节点使用
	- 客户端维护复杂：SDK和客户端本身的消耗(例如更多的连接池)
3. 很多场景下Redis Sentinel已经足够好了。
4. 选择的时候考虑到底是不是真的需要集群。(QPS)

---