---
title: Redis学习笔记（四）：消息订阅发布,事务,锁,基数统计及客户端命令
date: 2018-04-12 19:05:30
tags: Redis
categories: NoSQL

---
## 0.学习准备
1. 参考资料：
	参考视频：《动力节点Redis视频教程》
	参考教程：<http://www.redis.net.cn/tutorial/3501.html>
	Redis命令参考手册：
	<http://redisdoc.com/>
	w3cschoolRedis教程：<https://www.w3cschool.cn/redis/nma27f21.html>
2. 学习准备
开启redis服务

---
## 1.发布订阅及消息队列的理论知识
1. Redis的发布订阅是一种消息通信模式，发送者(pub)发送消息，订阅者(sub)接收消息。
发布订阅消息也叫生产者消费者模式，是实现消息队列的一种方式。
2. 示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-01.jpg)
三要素：生产者(producer)，消费者(consumer)，消息服务(broker)
3. 关系图：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-02.jpg)
可以很好的实现系统的解耦。
4. 对于Redis来说,Redis充当的就是这个消息服务器的作用。
但是主流的还是中间件：activeMQ，rabbitMQ

---
## 2.Redis消息订阅发布的实现
### 命令行实现
1. 开启四个客户端，一个用于发送消息，三个用于发送消息。
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-03.jpg)
第一个用于发送消息，后面三个用于接收消息。
2. 让三个客户端订阅某个消息主题：`subscribe channel`
channel是主题名字，可以任意指定。（这里指定的是chan1）
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-04.jpg)
3. 让那个客户端往发送消息：`publish channel message`
这个channe需要和刚刚订阅的channel完全一致，否则无法接受消息（chan1）。
message需要用引号括起来。
返回值：
接收到信息message的订阅者数量。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-05.jpg)
其他三个客户端几乎是同时接受到消息的。
需要先订阅，才能接收到消息，先发布不能接收到消息。
4. 如果订阅的是匹配的主题，可以使用`psubscribe chan*`
表示可以得到以chan开头的所有频道的消息。

### Jedis实现
1. 新建Message包，新建SubScriber类如下：
		public class Subscriber extends JedisPubSub{
			/**
			 * 接收到消息后的处理方法
			 * 回调方法。
			 */
			@Override
			public void onMessage(String channel, String message) {
				// TODO Auto-generated method stub
				super.onMessage(channel, message);
				System.out.println("从频道["+channel+"]得到消息："+message);
			}
			class sub extends BaseOperation{
				/*
				 * 订阅某一主题的方法
				 */
				public void RedisSubscribe(Subscriber subscriber,String channel){
					Jedis jed=getJedis();
					//订阅这一频道
					jed.subscribe(subscriber, channel);
				}
			}
		}
2. 新建发布者类publisher如下：
		public class Publisher extends BaseOperation{
			//发送消息的方法
			public void RedisPublish(String channel,String message){
				Jedis jed=getJedis();
				//发送
				jed.publish(channel, message);
			}
		}
3. 基本操作类如下(连接Jedis)：
		public class BaseOperation {
			public Jedis getJedis(){
				//连接Jedis服务器
				Jedis jed =new Jedis("10.186.151.205", 6379);
				//输入密码，否则只能登陆而不能获取数据
				jed.auth("123456");
				return jed;
			}
		}
4. 在src/test/java下新建message包，创建测试类如下：
		public class PubSubTest {
			Subscriber subscriber=new Subscriber();
			String chan="chan1";
		
			@Test
			public void testSub(){
				Sub ss1=subscriber.new Sub();
				Sub ss2=subscriber.new Sub();
				Sub ss3=subscriber.new Sub();
				ss1.RedisSubscribe(subscriber,chan); //一直停留在这里接收消息
				//这里阻塞了
				ss2.RedisSubscribe(subscriber,chan);
				ss3.RedisSubscribe(subscriber,chan);
			}
			@Test
			public void testPub(){
				Publisher pub=new Publisher();
				pub.RedisPublish(chan, "hello everyone");
			}
		}
5. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-06.jpg)
6. Jedis关于订阅的一些其他方法：
		public abstract class JedisPubSub {  
			//取消所有订阅  
			public void unsubscribe() {  
				...
			}
			
			//取消订阅频道
			public void unsubscribe(String... channels) {  
				...  
			}
			
			//增加订阅频道
			public void subscribe(String... channels) {  
				...
			}
		  
			//增加订阅频道的匹配表达式  
			public void psubscribe(String... patterns) {  
				...  
			}
		
			//取消所有按表达式的订阅  
			public void punsubscribe() {  
				...  
			}  
			
			//取消表达式匹配的频道  
			public void punsubscribe(String... patterns) {  
				...  
		 	}   
		}


### 总结
1. 发布订阅是消息队列的一种方式，基于消息队列的方式可以实现系统解耦，巅峰削谷，顶住流量洪峰等。
但是实时性不是很高，因为接收者会处理会有延时。
2. Redis目前的主要业务是键值对的存储，缓存等。消息队列只是Redis的一种尝试而已。可以持续关注Redis主要的发展。
3. 常用的消息队列（中间件）:
ActiveMQ,RabbitMQ

---
## 3.Redis事务
1. 和MySQL中的事务大致一样。
2. 事务是指一系列的操作步骤，要么全部执行，要么全部不执行。
要保证一系列的操作都成功就提出了事务控制的概念。
3. Redis中的事务：
Redis中的事务是一组命令的集合，至少是两个或两个以上的命令，要抱保证这些事务在执行时不会被其他命令打断。

### Redis对事务的实现
简单来说三步：开启事务-->执行命令-->提交/回滚事务
**1)正常情况：**
1. 开启事务：
`MULTI`
标记一个事务块的开始。
事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令原子性(atomic)地执行。
2. 添加命令到命令队列：
就和普通的执行命令一样。
3. `EXEC`
执行所有事务块内的命令。
假如某个(或某些) key 正处于 WATCH 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 EXEC 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。
（详见后面的Watch机制）
4. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-07.jpg)

**2)异常情况：语法错误**
1. `MULTI`：开启事务
`SET k1 v1`：正常命令
`SET k2`：语法错误
2. 这时使用`EXEC`提交事务：
则事务内所有的命令都不会执行。前面的k1也会设置失败。
3. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-08-1.jpg)
这里使用`SET k2`导致语法错误效果相同。

**3)例外情况及放弃事务**
1. 参数错误但是事务依旧提交
`MULTI`：开启事务
`SET k1 v1`：正常命令
`incr k1`：语法正确，但是类型错误，字符串不能自增
`EXEC`：事务仍然提交了
2. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-09.jpg)
所以Redis是部分支持事务。
3. 如果入队列时报错，则会成功提交事务，如果入队列时失败了，则该操作会回滚。
提交事务时，队列中的操作能执行就执行，不能执行就报错。
4. 可以放弃事务：`discard`
放弃事务则整个事务命令队列中的命令都不执行。

**4)复杂情况：和锁联系在一起**
1. 详见后面的watch机制实现乐观锁。

---
## 4.悲观锁，乐观锁及简单实现
**1)悲观锁，乐观锁的简介：**
1. **悲观锁**(Pessimistic Lock)：
顾名思义，就是很悲观的锁，每次去拿数据的时候都认为别人会修改数据。所以每次在拿数据时都会先上锁。这样别人想拿这个数据就会先上锁，别人想拿数据时就会阻塞，直到它拿到锁。
传统的关系型数据库里面用到了很多这种锁机制，如行锁，表锁，读锁，写锁等。都是在做操作之前先上锁，让别人无法操作该数据。
2. **乐观锁**（Optimistic Lock）：
使用的比较多，每次去拿数据的时候都认为别人不会修改数据，所以不会上锁。但是在更新时会判断以下再次期间别人有没有去更新这条数据。一般用版本号机制做判断。
适用于多读的数据类型，这样可以提高吞吐量。
3. 乐观锁的实现原理：
乐观锁的大多数情况是基于数据版本号的机制实现的。为每一个数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是在数据库表中增加一个version字段，来实现读取数据时将数据版本号一同读出。之后更新时将版本号加1。此时将该版本号和数据库中已有的版本号进行比对，如果更新的版本号较大则更新数据，否则不予更新。
4. 乐观锁及被悲观锁的选择：

5. 简单的例子的示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-10.jpg)

**2)Watch机制实现乐观锁：**
1. `watch`方法：
可以监视一个或者多个key。
监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
2. `unwatch`方法：
取消WATCH命令对所有key的监视。
如果在执行WATCH命令之后，EXEC命令或DISCARD命令先被执行了的话，那么就不需要再执行UNWATCH了。
3. 假如某个(或某些)key正处于WATCH命令的监视之下，且事务块中有和这个(或这些)key相关的命令，那么EXEC命令只在这个(或这些)key没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。
4. . 测试命令：(有两个客户端)
客户端1：`set k1 111`
客户端1：`watch k1`  -->监视k1
客户端2：`set k1 222`  -->更改k1
客户端1：`MULTI`  -->开启事务
客户端1：`set k1 333`  -->更改k1
客户端1：`EXEC`  -->提交事务
客户端1：`get k1`  -->查看更改
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-11.jpg)

---
## 5.基数统计HyperLogLog
**1)HyperLogLog简介：**
1. Redis HyperLogLog是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。是Redis存储的第六种数据类型。
2. 在Redis里面，每个HyperLogLog键只需要花费12KB内存，就可以计算接近2^64个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比
3. 因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以HyperLogLog不能像集合那样，返回输入的各个元素。
只做统计，不做存储。
4. 基数统计与基数估计：
基数就是不重复的元素。
比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数为5。 基数估计就是在误差可接受的范围内，快速计算基数。

**2)HyperLogLog常用方法：**
1. `Pfadd`命令
`PFADD key element [element ...]`
将所有元素参数添加到HyperLogLog类型数据结构中。
如果有元素被添加，则返回1，否则返回0。
2. `Pfcount`命令
`PFCOUNT key [key ...]`
返回给定key的HyperLogLog的基数估算值。
3. `Pfmerge`命令
`PFMERGE destkey sourcekey [sourcekey ...]`
将多个HyperLogLog合并为一个HyperLogLog，合并后的HyperLogLog的基数估算值是通过对所有 给定HyperLogLog进行并集计算得出的。
destkey是合并之后的key,sourcekey是合并之前的各个key。
4. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-12.jpg)

---
## 6.客户端命令：
**1)关于客户端连接：**
1. Redis通过监听一个TCP端口或者Unix socket的方式来接收来自客户端的连接，当一个连接建立后，Redis内部会进行以下一些操作：
	- 首先，客户端socket会被设置为非阻塞模式，因为Redis在网络事件处理上采用的是非阻塞多路复用模型。
	- 然后为这个 socket 设置 TCP_NODELAY 属性，禁用Nagle算法
	- 然后创建一个可读的文件事件用于监听这个客户端socket的数据发送
2. 最大连接数：
可以在redis.conf中进行配置，maxclients的默认值是10000。

**2)常用的客户端命令：**
1. `client list`： 返回连接到redis服务的客户端列表
`client list`的返回值详解：
	>命令返回多行字符串，这些字符串按以下形式被格式化：
	每个已连接客户端对应一行（以 LF 分割）
	每行字符串由一系列 属性=值 形式的域组成，每个域之间以空格分开
	以下是域的含义：
		addr ： 客户端的地址和端口
		fd ： 套接字所使用的文件描述符
		age ： 以秒计算的已连接时长
		idle ： 以秒计算的空闲时长
		flags ： 客户端 flag （见下文）
		db ： 该客户端正在使用的数据库 ID
		sub ： 已订阅频道的数量
		psub ： 已订阅模式的数量
		multi ： 在事务中被执行的命令数量
		qbuf ： 查询缓冲区的长度（字节为单位， 0 表示没有分配查询缓冲区）
		qbuf-free ： 查询缓冲区剩余空间的长度（字节为单位， 0 表示没有剩余空间）
		obl ： 输出缓冲区的长度（字节为单位， 0 表示没有分配输出缓冲区）
		oll ： 输出列表包含的对象数量（当输出缓冲区没有剩余空间时，命令回复会以字符串对象的形式被入队到这个队列里）
		omem ： 输出缓冲区和输出列表占用的内存总量
		events ： 文件描述符事件（见下文）
		cmd ： 最近一次执行的命令
	客户端 flag 可以由以下部分组成：
		O ： 客户端是 MONITOR 模式下的附属节点（slave）
		S ： 客户端是一般模式下（normal）的附属节点
		M ： 客户端是主节点（master）
		x ： 客户端正在执行事务
		b ： 客户端正在等待阻塞事件
		i ： 客户端正在等待 VM I/O 操作（已废弃）
		d ： 一个受监视（watched）的键已被修改， EXEC 命令将失败
		c : 在将回复完整地写出之后，关闭链接
		u : 客户端未被阻塞（unblocked）
		A : 尽可能快地关闭连接
		N : 未设置任何 flag
	文件描述符事件可以是：
		r : 客户端套接字（在事件 loop 中）是可读的（readable）
		w:客户端套接字（在事件 loop 中）是可写的（writeable）
2. `client list`测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-13.jpg)
3. `client setname`：设置当前连接的名称
这个名字会显示在`CLIENT LIST`命令的结果中， 用于识别当前正在与服务器进行连接的客户端。
为了避免和 `CLIENT LIST` 命令的输出格式发生冲突，名字里不允许使用空格。
要移除一个连接的名字， 可以将连接的名字设为空字符串 "" 。
默认是没有名字的nil。
4. `client getname`：获取通过CLIENT SETNAME命令设置的服务名称
因为新创建的连接默认是没有名字的，对于没有名字的连接，`CLIENT GETNAME`返回空白回复nil。
5. `client kill`：关闭客户端连接
6. `client pause time`：挂起当前客户端连接，time指定挂起的时间以毫秒计
测试pause挂起：
![](http://p5ki4lhmo.bkt.clouddn.com/00037Redis%E5%AD%A6%E4%B9%A04-14.jpg)
挂起后执行下一个命令时会阻塞，直到挂起结束(这里是20秒)才继续执行。

---
## 7.命令总结
**1)发布订阅：**
`subscribe channel`： 订阅一个主题端口
`publish channel message`： 向某个主题发送消息
`psubscribe chan*`： 订阅以chan开头的所有主题的信息

**2)事务及锁：**
`MULTI`： 开启事务
`EXEC`： 提交事务
`discard`： 取消事务
`watch key [key...]`： 可以监视一个或者多个key。
`unwatch key [key...]`： 取消监视一个或者多个key。

**3)基数统计：**
`PFADD key element [element ...]`： 将所有元素参数添加到HyperLogLog类型数据结构中。
`PFCOUNT key [key ...]`： 返回给定key的HyperLogLog的基数估算值。
`PFMERGE destkey sourcekey [sourcekey ...]`
将多个HyperLogLog合并为一个HyperLogLog，原本的也不会消失。

**4)客户端方法：**
`client list`： 返回连接到 redis 服务的客户端列表
`client setname`： 设置当前连接的名称
`client getname`： 获取通过CLIENT SETNAME命令设置的服务名称
`client kill`： 关闭客户端连接
`client pause time`： 挂起当前客户端连接，time指定挂起的时间以毫秒计

---

