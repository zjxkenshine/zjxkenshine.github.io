---
title: Redis学习笔记（八）：Redis Cluster集群简单搭建
date: 2018-4-25 11:51:56  
tags: 
- Redis
- Ruby
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
	- 呼唤集群
	- 数据分布
	- 集群原生搭建
	- 使用Ruby工具搭建

---
## 1.呼唤集群
1. 为什么要呼唤集群：
	- 并发量不足：默认是10万QPS(没有太多慢查询的情况下),如果超过10万就需要集群。
	- 数据量不足：单台机器的存储容量不足。
	- 网络流量的需求过大。
2. 解决方案：
	- 早期的解决方案，配置更强大的机器。(很贵)
	- 正确的解决方案，分布式集群。简单解释就是加机器。
3. Redis在3.0的时候提供了分布式的支持。	
4. 在集群中只有db0,没有db1到db15这些数据库。

---
## 2.数据分布
**1)数据分区：**
1. 数据分区示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-01.jpg)
但是如何去设计一个分区是非常重要的。
2. 主要的分布方式：
顺序分布，哈希分布(使用哈希函数将数据打散但是均匀分布)
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-02.jpg)
3. 两种数据分布方式的区别：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-03.jpg)
注意上图有误的一个地方，顺序分布是不支持批量操作的。
4. 因为是Redis Cluster,所以主要介绍Hash分布，主要有三种分区方式：(前两种是客户端的分区)
	- 节点取余分区
	- 一致性哈希分区
	- 虚拟槽分区(Redis Cluster的分区方式)

**2)节点取余分区：**
1. hash(key)%nodes
nodes是节点数量，将数据放到余数对应的节点中。
如：
三个节点：hash(key)%3
四个节点：hash(key)%4
2. 有一个问题，如果需要增加一个节点：
如从3个节点变成4个节点，那么几乎所有的数据都需要进行回写，数据迁移率达80%。
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-04.jpg)
回写：访问某节点无数据，重新查询并写入缓存。
3. 所以这种节点取余分区建议使用多倍扩容的方式：
使用翻倍扩容，如2个变为4个，3个变6个，则数据回写会降低为50%：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-05.jpg)
4. 迁移数据之后是无法从缓存中读取到的，所以会从数据库查询之后再进行缓存的回写，大量的回写是很不正常的。
5. 总结：
	- 客户端分片：哈希+取余
	- 缺点：节点伸缩导致数据大量回写
	- 最好使用翻倍扩容
	- 是一些很早的分布式缓存的使用方式

**3)一致性哈希分区：**
1. 在memcache中使用广泛。
2. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-06.jpg)
key进行哈希计算后落在某一个位置，然后进行顺时针查找节点并存入。
3. 如果再添加一个节点，则示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-07.jpg)
仍然有数据回写，但是影响范围非常小，只影响了node1和node2之间的部分数据。
尤其是在节点非常多的情况下，效果非常好。
4. 这种方式也是一种数据回写的过程，原来在n2中的部分数据就变成垃圾了，客户端访问node5,没有数据，进行数据回写。不会将原有的数据迁移到新节点。
5. 一致性哈希总结：
	- 客户端分片：哈希+顺时针(优化取余)
	- 节点伸缩：只影响临近节点，但还是有数据回写
	- 翻倍伸缩：可以保证最小迁移数据和负载均衡

**4)虚拟槽分区：**
1. Redis Cluster的方式
2. 特点：
	- 预设虚拟槽，每一个槽映射一个数据子集，一般比节点大
	- 良好的哈希函数，如CRC16等
	- 服务端管理节点，槽，数据
3. 虚拟槽方式示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-08.jpg)
说明：
如图将槽范围分为了5份分给5个节点，然后key进行CRC16哈希函数操作后算出一个值，然后按照该值去选择节点保存。
客户端查数据时key会访问一个节点，节点之间共享消息，如果选择节点不是对应节点，那个节点会返回对应的节点号。
Redis Cluster默认指定的槽的数量就是16384。
4. 增加新节点时从现有的节点分配，并且将数据也一起分配
5. 暂时不用了解那么清楚，因为后面Redis Cluster都是虚拟槽分区的运用

---
## 3.集群架构与安装架构
**1)集群架构简介：**
1. 单机架构示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-09.jpg)
2. 分布式架构示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-10.jpg)
每个节点之间是通过GOSSIP协议相互通信的。可以互相感知到对应节点所负责的槽，且每个节点都知道哪个槽是交给哪个节点负责的。而且客户端也需要知道。


**2）Redis Cluster的安装架构：(不包括节点伸缩)** 
1. 主要元素：
	- 节点：nodes
	- meet操作：节点间相互通信的基础
	- 指派槽：优先访问该槽
	- 复制：保证高可用(每个主节点都有一个备节点)
	但是这种时候的内部监控不是依赖Sentinel，而是通过他们节点之间的交互。
2. 主要元素的实现：
*节点*：通过配置`cluster-enabled：yes`,来配置是否是集群的模式。
*meet*：AmeetB则AB可以进行通信，相当于PING
而如果AmeetB，BmeetC,AC也可以进行通信(gossip协议)
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-11.jpg)
*指派槽*：对应可以访问过来时有数据返回数据，无数据返回对应的槽的节点
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-12.jpg)
3. RedisCluster的特定：
	- 主从复制
	- 高可用
4. 安装方式：
	- 原生命令安装：节点，meet,分配槽，搭配主从
	- 官方工具安装：Ruby工具一键安装，只能实现简单配置
5. 理解安装过程：
	其实就是上述安装框架的四个步骤

---
## 4.Redis Cluster原生命令的安装方式
**1)RedisCluster源生命令安装：简单过程**
1. 配置开启节点：配置模板(集群需要的最简配置)
		port ${port}
		daemonize yes
		dir "/opt/redis/redis/data"  //目录，可以随意指定
		logfile "${port}.log"
		dbfilename "dump-${port}.rdb"
		//是否开启集群
		cluster-enabled yes
		//节点配置文件
		cluster-config-file nodes-${port}.conf
最好配置六个，三对主从。
2. Cluster节点的主要配置：
		cluster-enabled yes
		cluster-node-timeout 15000  //很多作业，可以简单认为是故障转移时间
		cluster-config-file "node.file" //节点配置文件(单机多部署则用端口号区分),用于记录集群信息
		cluster-require-full-coverage no //一个节点坏掉则对外不提供服务,默认yes,一般配置为no
3. 开启节点：
		# redis-server redis-port.conf
4. 使用meet使各个节点产生通信：
		cluster meet ip port
如有六个节点则最少要执行五次meet操作。
5. 分配槽：
		cluster addslots slot[slot]
一共16384个节点，而有6个三组节点(7001~7006)，所以这样分：
0~5461一组分配给7001节点，
5462~10922一组分配给7002节点，
10923~16383一组分配给7003节点。
6. 判断集群是否设置成功，并可以对外提供服务。
7. 设置主从关系：
		cluster replicate node-id
node-id指的是节点的id,在节点启动时会自动分配。
node-id和run-id是不同的，它不会因为节点重启而改变。
8. 原生安装了解过程就可以，不必精通

**2)配置节点：**
1. 配置redis-7000.conf：集群的最低配置
		port 7000
		daemonize yes
		dir "./"
		logfile "node7000.log"
		dbfilename "dump7000.rdb"
		cluster-enabled yes
		cluster-node-timeout 15000
		cluster-config-file nodes-7000.conf
		cluster-require-full-coverage no
		//所有节点密码需要一致
		masterauth 123456
		requirepass 123456
配置图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-13-1.jpg)
2. 快速生成其他5个节点的配置：
`# sed 's/7000/7001/g' redis-7000.conf > redis-7001.conf`
以此类推，生成7000~7005这六个节点
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-14.jpg)
3. 注意设置了密码的情况一定要设置
		masterauth password
		requirepass password
这两项，而且每个节点的密码也最好相同。可以像这样直接在配置文件中配置的。但是这样需要重启Redis服务，这里我还没启动所以这样配置。
如果要动态配置则可以在该端口客户端使用如下命令配置：
		config set masterauth passwd123
		config set requirepass passwd123
		config rewrite 
这样就可以不用重启进行配置了。
4. 启动这些节点的redis服务：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-15.jpg)
后面有个cluster说明是集群节点。
5. 注意如果节点很多的话可以写个脚本来完成。
6. 如果这时候使用客户端并执行某些操作(如get,set)，会抛出一个错误：
		(error)CLUSTERDOWN The cluster is down
这表明当前集群时下线的，不可用的。因为现在仅仅开启了Redis服务，集群还没有搭建好。
7. 可以在客户端下执行以下命令查看一些信息：
`cluster nodes`：查看配置信息，交互节点信息
`cluster info`：查看集群信息(状态，是否分配槽等)
如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-16.jpg)

**3)配置meet节点交互(节点握手)：**
1. 语法：在客户端下
		cluster meet ip port
ip就是想握手的节点的ip,port就是端口。
最好从单一节点的客户端出发，这样到其他节点只需要执行n-1次握手。
2. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-17.jpg)
重复握手并不报错。
3. 在客户端执行`cluster nodes`命令可以查看与该节点达成交互的节点：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-18.jpg)
说明整个集群已经达到了一种互通的效果。
还可以执行`cluster info`命令查看一些其他信息。
4. 但是此时，集群没有分配槽，依然不可以进行读写：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-19.jpg)

**4)分配槽：**
1. 在需要分配槽的节点客户端下执行命令：
		cluster addslots 0
这样就可以将0这个槽分配给这个端口的节点。
但是如果要分配0~5461个槽分配到这个端口，就需要一个一个进行分配，这时就需要使用shell脚本了。
2. 编写shell脚本addslots.sh如下：
		# 起始终止slots
		start=$1
		end=$2
		# 端口
		port=$3
		for slot in `seq ${start} ${end}`
		do
		        redis-cli -p ${port} -a "123456" cluster addslots ${slot} 
		done
3. 首先确定好主从关系再进行分配槽：
假设7000，7001，7002是主节点，
7003，7004，7005是对应的从节点。
4. 根据确定的主从关系来执行上述shell脚本:
		sh addslots.sh 0 5461 7000
		sh addslots.sh 5462 10922 7001
		sh addslots.sh 10923 16383 7002
要执行好长好长好长时间(绝不夸张)。
仅了解这么做就行了，直到原理，实际生产过程中不会使用原生方式的，即容易出错，又耗费时间。
5. 若仅分配完一部分槽，集群的状态依然是失败的。
只有当所有的槽都分配完毕了才算成功。
6. 集群完毕后随便使用哪个客户端，使用`cluster info`以及`cluster nodes`命令查看节点状态：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-20.jpg)
已经OK了，说明这时就可以使用读写命令了。
7. 但是还需要一步：主从复制
8. 补充：还可以使用`cluster slots`查看槽的分配信息及主从关系。

**5)配置主从关系：**
1. 命令：在从节点下(7003,7004,7005)客户端执行
		cluster replicate node-id
这个node-id可以使用cluster nodes看到，就是ip前的那一大串。
也可以直接使用：
		redis-cli -p 从节点端口 -a "密码" cluster replicate nodeid
来执行客户端命令。
2. 操作如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-21.jpg)
操作成功。
3. 使用`cluster slots`及`cluster info`查看槽及主从信息：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-22.jpg)
4. 这样子就大功告成啦。

**6)如何使用：**
1. 客户端命令：
`redis-cli -c -h ip -p port`
在使用时会根据所在的槽口自动进行节点的跳转。
如果不加-c参数则跳转时会报错，如下：
`(error) MOVED 12182 172.0.0.1:7002`
2. 搭建好的集群，所有服务关闭后重启不需要重新搭建
3. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-23.jpg)
4. 注意一个很重要的问题：
一定要在第一次连接客户端时使用参数-a来输密码，而不是进去之后再输，否则就会出现这种情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-24.jpg)
卡了我一晚上。。。而且查遍全网也查不出问题。。。

---
## 5.在Linux下安装Ruby环境
**1)查看是否安装Ruby**
1. 使用Ruby工具搭建Redis集群需要使用redis-trib.rb
2. redis-trib.rb安装(针对Redis集群的)简单分为三步：
	- 下载，编译，安装Ruby
	- 安装rubygem redis(ruby对Redis的客户端)
	- 安装redis-trib.rb
3. 首先查看是否安装了Ruby:
		rpm -qa | grep ruby
		yum list | grep ruby
或者直接`man ruby`或者`ruby -v`
很幸运，我的安了，但是默认都是安装的Ruby2.0.0,想要安装更高版本。
4. 下面介绍安装高版本Ruby的三种方式。

**2)方式一：手动下载解压编译安装**
1. 下载并解压Ruby：
过程和Redis的非常相似
下载：
		# wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz
或者在Windows中下载后传过去。
如果没有用yum，则需要解压，编译(make)，安装(make install)。
解压：
		# tar -zxvf ruby-2.5.1.tar.gz 
2. 移动到Ruby目录：
跳转到ruby2.5.1
		# cd ruby2.5.1
3. 配置，编译，安装
源码安装一般有这三个步骤：配置(configure)、编译(make)、安装(make install)
配置：
		# ./configure  --prefix=/opt/ruby
编译：(时间可能有点长)
		# make
安装：
		# make install
这时Ruby就安装到了/opt/ruby目录下了。
4. 使用软连接或者cp复制将系统自带的Ruby相关命令覆盖：
为什么要覆盖呢：
跳转到/opt/ruby目录执行如下操作：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-25.jpg)
发现默认的ruby命令还是系统自带的。
(注意另个-f参数都是强制替换，如果不想替换掉原来的建议换名字)
可以使用软连接覆盖/usr/bin中的可执行文件：
		# ln -sf /opt/ruby/bin/ruby /usr/bin/ruby
		# ln -sf /opt/ruby/bin/gem /usr/bin/gem
也可以使用cp覆盖：
		# cp -f /opt/ruby/bin/ruby /usr/bin/ruby
		# cp -f /opt/ruby/bin/gem /usr/bin/gem
替换完毕再执行：ruby -v
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-26-1.jpg)

**3)方式二：切换yum安装源安装**
1. 使用yum安装的Ruby默认也是2.0.0版本的
2. 具体安装步骤如下：
切换安装源：
		# yum install centos-release-scl-rh　　　　//会在/etc/yum.repos.d/目录下多出一个CentOS-SCLo-scl-rh.repo源
安装并配置：
		# yum install rh-ruby23 -y
		# scl enable rh-ruby23 bash
3. 这几步执行完毕就可以了。未测试。

**4)方式三：RVM 安装(脚本安装)**
1. 先执行一条官方 https://rvm.io/ 复制来的长命令（...C0E3空格7D2B...）：
		# gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
2. 然后执行命令(连接)：
		# \curl -sSL https://get.rvm.io | bash -s stable
3. 进行安装：
按提示执行：
		# source  /etc/profile.d/rvm.sh
查看哪些可安装版本：
		# rvm list known
安装某版本ruby,直接跟版本号即可：
		# rvm install 2.5.1
4. 使用`ruby-v`,`gem-v`查看是否安装成功。
5. 更加具体生动的(带图片的)过程可以访问大佬博客：
<https://www.cnblogs.com/ding2016/p/7903147.html>

---
## 6.使用Ruby工具搭建集群
**1)安装redis-trib.rb：**
1. redis-trib.rb安装简单分为三步：(上面已经安装了Ruby环境了)
	- 下载，编译，安装Ruby
	- 安装rubygem redis(ruby对Redis的客户端)
	- 安装redis-trib.rb
2. 安装rubygem redis客户端
		# gem install  redis --version 4.0.0
或者使用wget来下载安装：
		# wget http://rubygems.org/downloads/redis-4.0.0.gem
		# gem install -l redis-4.0.0.gem
如果gem install报错，可以查看Redis解决方案或者博客：
<https://blog.csdn.net/feinifi/article/details/78251486>
安装成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-27.jpg)
确认一下：
		# gem list -- check redis gem
可以查看本机所安装的包：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-28.jpg)
3. 查看是否有redis-trib.rb：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-29.jpg)
4. 查看是否能使用该命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-30.jpg)
create是搭建一个集群
reshard是集群扩展
rebalance是负载均衡
5. 设置软连接到/usr/bin：
		# ln -sf /usr/local/redis/src/redis-trib.rb /usr/local/bin/redis-trib.rb
6. 使用redis-trib.rb看看是否成功
能弹出和上面一样的用法提示说明成功了
7. 下面就可以使用该命令去自动化搭建集群了

**2)第一步：配置开启几个Redis服务(8000-8005)**
1. 创建8000节点配置文件redis-8000.conf配置如下：
		port 8000  
		cluster-enabled yes  
		cluster-config-file "nodes-8000.conf"  
		cluster-node-timeout 15000  
		dir "./"  
		appendonly yes  
		appendfilename "appendonly-8000.aof"  
		logfile "node-8000.log"  
		daemonize yes  
		pidfile "/var/run/redis-8000.pid"  
		dbfilename "dump-8000.rdb"  
		cluster-require-full-coverage no
		masterauth 123456
		requirepass 123456
其余节点相同：
`# sed 's/8000/8001-5/g' redis-7000.conf > redis-8001-5.conf`
具体的创建过程就不截图了。
2. 开启这些服务：
		# redis-server redis-8000.conf
		# redis-server redis-8001.conf
		# redis-server redis-8002.conf
		# redis-server redis-8003.conf
		# redis-server redis-8004.conf
		# redis-server redis-8005.conf
查看进程：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-31.jpg)
3. 如果像我一样配置了密码，你还需要以下几步进行配置：
使用`gem env`查看gem path:这是你的软件安装路径
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-32.jpg)
可以看到我的路径(第一个)是/opt/开头的，那么久修改它：
		# vim ${你的地址}/gems/redis-4.0.0/lib/redis/client.rb
修改password：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-33.jpg)

**3)第二步，一键搭建，完成了**
1. 一键搭建Redis集群：
		# redis-trib.rb create --replicas 1 127.0.0.1:8000 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 127.0.0.1:8004 127.0.0.1:8005
--replicas后面的`1`代表每个主节点有一个从节点。
如果太多节点还是写个shell脚本吧。
2. 中间可能需要输入一个yes确认：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-34.jpg)
创建完成：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-35.jpg)
3. 测试是否搭建成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00049Redis%E5%AD%A6%E4%B9%A08-36.jpg)
牛逼，大功告成。
4. 注意，如果节点非常多的话，这种方式也不是一个非常好的方式，最好搭建一个云平台，来对各种数据进行管理。我看《Redis运行与维护》那本书上有讲，有空买一本学习学习。

---