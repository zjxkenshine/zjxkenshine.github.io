---
title: Redis学习笔记（四）：Redis消息订阅发布，事务及锁
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

### 总结


---
