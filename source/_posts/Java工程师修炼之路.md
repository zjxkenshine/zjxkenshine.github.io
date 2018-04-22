---
title: Java工程师修炼之路
date: 2018-4-22 21:50:03 
tags: 学习路线
categories: 杂谈

---
# 一、基础篇
---
## JVM
<br>
### JVM内存结构
堆、栈、方法区、直接内存、堆和栈区别
<br>
### Java内存模型
内存可见性、重排序、顺序一致性、volatile、锁、final
<br>
### 垃圾回收
内存分配策略、垃圾收集器（G1）、GC算法、GC参数、对象存活的判定 
<br>
### JVM参数及调优
<br>
### Java对象模型
oop-klass、对象头
<br>
### HotSpot
即时编译器、编译优化
<br>
### 类加载机制
classLoader、类加载过程、双亲委派（破坏双亲委派）、模块化（jboss modules、osgi、jigsaw）
<br>
### 虚拟机性能监控与故障处理工具
jps, jstack, jmap、jstat, jconsole, jinfo, jhat, javap, btrace、TProfiler
<br>

---
## 编译与反编译
javac 、javap 、jad 、CRF
<br>

---
## Java基础知识
<br>
### 阅读源代码
String、Integer、Long、Enum、BigDecimal、ThreadLocal、ClassLoader & URLClassLoader、ArrayList & LinkedList、 HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap、HashSet & LinkedHashSet & TreeSet
<br>
### Java中各种变量类型
<br>
### 熟悉Java String的使用，熟悉String的各种函数
JDK 6和JDK 7中substring的原理及区别、
replaceFirst、replaceAll、replace区别、
String对“+”的重载、
String.valueOf和Integer.toString的区别、
字符串的不可变性
<br>
### 自动拆装箱
Integer的缓存机制
<br>
### 熟悉Java中各种关键字
transient、instanceof、volatile、synchronized、final、static、const 原理及用法。
<br>
### 集合类
常用集合类的使用
ArrayList和LinkedList和Vector的区别 
SynchronizedList和Vector的区别
HashMap、HashTable、ConcurrentHashMap区别
Java 8中stream相关用法
apache集合处理工具类的使用
不同版本的JDK中HashMap的实现的区别以及原因
<br>
### 枚举
枚举的用法、枚举与单例、Enum类
<br>
### Java IO&Java NIO，并学会使用
bio、nio和aio的区别、三种IO的用法与原理、netty、mina
<br>
### Java反射与javassist
反射与工厂模式、`java.lang.reflect.*`
<br>
### Java序列化
什么是序列化与反序列化、为什么序列化
序列化底层原理
序列化与单例模式
protobuf
为什么说序列化并不安全
<br>
### 注解
元注解、自定义注解、Java中常用注解使用、注解与反射的结合
<br>
### JMS
什么是Java消息服务、JMS消息传送模型
<br>
### JMX
`java.lang.management.*`、`javax.management.*`
<br>
### 泛型
泛型与继承
类型擦除
泛型中K T V E  
object等的含义、泛型各种用法
<br>
### 单元测试
junit、mock、mockito、内存数据库（h2）
<br>
### 正则表达式
`java.lang.util.regex.*`
<br>
### 常用的Java工具库
commons.lang, commons.*... guava-libraries netty
<br>
### 什么是API&SPI
<br>
### 异常
异常类型、正确处理异常、自定义异常
<br>
### 时间处理
时区、时令、Java中时间API
<br>
### 编码方式
解决乱码问题、常用编码方式
<br>
### 语法糖
Java中语法糖原理、解语法糖
<br>

---
## Java并发编程
<br>
### 什么是线程，与进程的区别
<br>
### 阅读源代码，并学会使用
Thread、Runnable、Callable、ReentrantLock、ReentrantReadWriteLock、Atomic*、Semaphore、CountDownLatch、、ConcurrentHashMap、Executors
<br>
### 线程池
自己设计线程池、submit() 和 execute()
<br>
### 线程安全
死锁、死锁如何排查、Java线程调度、线程安全和内存模型的关系
<br>
### 锁
CAS、乐观锁与悲观锁、数据库相关锁机制、分布式锁、偏向锁、轻量级锁、重量级锁、monitor、锁优化、锁消除、锁粗化、自旋锁、可重入锁、阻塞锁、死锁
<br>
### 死锁
<br>
### volatile
happens-before、编译器指令重排和CPU指令重
<br>
### synchronized
synchronized是如何实现的？
synchronized和lock之间关系
不使用synchronized如何实现一个线程安全的单例
<br>
### sleep 和 wait
<br>
### wait 和 notify
<br>
### notify 和 notifyAll
<br>
### ThreadLocal
<br>
### 写一个死锁的程序
<br>
### 写代码来解决生产者消费者问题
<br>
### 守护线程
守护线程和非守护线程的区别以及用法
<br>

---
# 二、 进阶篇
---
## Java底层知识
<br>
### 字节码、class文件格式
<br>
### CPU缓存，L1，L2，L3和伪共享
<br>
### 尾递归
<br>
### 位运算
用位运算实现加、减、乘、除、取余
<br>

---
## 设计模式
<br>
### 了解23种设计模式
<br>
### 会使用常用设计模式
单例、策略、工厂、适配器、责任链。
<br>
### 实现AOP
<br>
### 实现IOC
<br>
### 不用synchronized和lock，实现线程安全的单例模式
<br>
### nio和reactor设计模式
<br>

---
## 网络编程
<br>
### tcp、udp、http、https等常用协议
三次握手与四次关闭、流量控制和拥塞控制、OSI七层模型、tcp粘包与拆包
<br>
### http/1.0 http/1.1 http/2之前的区别
<br>
### Java RMI，Socket，HttpClient
<br>
### cookie 与 session
cookie被禁用，如何实现session
<br>
### 用Java写一个简单的静态文件的HTTP服务器
>实现客户端缓存功能，支持返回304 实现可并发下载一个文件 使用线程池处理客户端请求 使用nio处理客户端请求 支持简单的rewrite规则 上述功能在实现的时候需要满足“开闭原则”

<br>
### 了解nginx和apache服务器的特性并搭建一个对应的服务器
<br>
### 用Java实现FTP、SMTP协议
<br>
### 进程间通讯的方式
<br>
### 什么是CDN？如果实现？
<br>
### 什么是DNS？
<br>
### 反向代理
<br>

---
## 框架知识
<br>
### Servlet线程安全问题
<br>
### Servlet中的filter和listener
<br>
### Hibernate的缓存机制
<br>
### Hiberate的懒加载
<br>
### Spring Bean的初始化
<br>
### Spring的AOP原理
<br>
### 自己实现Spring的IOC
<br>
### Spring MVC
<br>
### Spring Boot2.0
Spring Boot的starter原理，自己实现一个starter
<br>
### Spring Security
<br>

---
## 应用服务器
<br>
### JBoss
<br>
### tomcat
<br>
### jetty
<br>
### Weblogic
<br>

---
## 工具
<br>
### git & svn
<br>
### maven & gradle
<br>

---
# 三、 高级篇
---
## 新技术
<br>
### Java 8
lambda表达式、Stream API、
<br>
### Java 9
Jigsaw、Jshell、Reactive Streams
<br>
### Java 10
局部变量类型推断、G1的并行Full GC、ThreadLocal握手机制
<br>
### Spring 5
响应式编程
<br>
### Spring Boot 2.0
<br>

---
## 性能优化
使用单例、使用Future模式、使用线程池、选择就绪、减少上下文切换、减少锁粒度、数据压缩、结果缓存
<br>

---
## 线上问题分析
<br>
### dump获取
线程Dump、内存Dump、gc情况
<br>
### dump分析
分析死锁、分析内存泄露
<br>
### 自己编写各种outofmemory，stackoverflow程序
HeapOutOfMemory、 Young OutOfMemory、MethodArea OutOfMemory、ConstantPool OutOfMemory、DirectMemory OutOfMemory、Stack OutOfMemory Stack OverFlow
<br>
### 常见问题解决思路
内存溢出、线程死锁、类加载冲突
<br>
### 使用工具尝试解决以下问题，并写下总结
当一个Java程序响应很慢时如何查找问题、
当一个Java程序频繁FullGC时如何解决问题、
如何查看垃圾回收日志、
当一个Java应用发生OutOfMemory时该如何解决、
如何判断是否出现死锁、
如何判断是否存在内存泄露
<br>

---
## 编译原理知识
<br>
### 编译与反编译
<br>
### Java代码的编译与反编译
<br>
### Java的反编译工具
<br>
### 词法分析，语法分析（LL算法，递归下降算法，LR算法），语义分析，运行时环境，中间代码，代码生成，代码优化
<br>

---
## 操作系统知识
<br>
### Linux的常用命令
<br>
### 进程同步
<br>
### 缓冲区溢出
<br>
### 分段和分页
<br>
### 虚拟内存与主存
<br>

---
## 数据库知识
<br>
### MySql 执行引擎
<br>
### MySQL 执行计划
如何查看执行计划，如何根据执行计划进行SQL优化
<br>
### SQL优化
<br>
### 事务
事务的隔离级别、事务能不能实现锁的功能
<br>
### 数据库锁
行锁、表锁、使用数据库锁实现乐观锁、
<br>
### 数据库主备搭建
<br>
### binlog
<br>
### 内存数据库
h2
<br>
### 常用的nosql数据库
redis、memcached
<br>
### 分别使用数据库锁、NoSql实现分布式锁
<br>
### 性能调优
<br>

---
## 数据结构与算法知识
<br>
简单的数据结构
栈、队列、链表、数组、哈希表、
<br>
树
二叉树、字典树、平衡树、排序树、B树、B+树、R树、多路树、红黑树
<br>
排序算法
各种排序算法和时间复杂度 深度优先和广度优先搜索 全排列、贪心算法、KMP算法、hash算法、海量数据处理
<br>

---
## 大数据知识
<br>
### Zookeeper
基本概念、常见用法
<br>
### Solr，Lucene，ElasticSearch
在linux上部署solr，solrcloud，，新增、删除、查询索引
<br>
### Storm，流式计算，了解Spark，S4
在linux上部署storm，用zookeeper做协调，运行storm hello world，local和remote模式运行调试storm topology。
<br>
### Hadoop，离线计算
HDFS、MapReduce
<br>
### 分布式日志收集flume，kafka，logstash
<br>
### 数据挖掘，mahout
<br>

---
## 网络安全知识
<br>
### 什么是XSS
XSS的防御
<br>
### 什么是CSRF
<br>
### 什么是注入攻击
SQL注入、XML注入、CRLF注入
<br>
### 什么是文件上传漏洞
<br>
### 加密与解密
MD5，SHA1、DES、AES、RSA、DSA
<br>
### 什么是DOS攻击和DDOS攻击
memcached为什么可以导致DDos攻击、什么是反射型DDoS
<br>
### SSL、TLS，HTTPS
<br>
### 如何通过Hash碰撞进行DOS攻击
<br>
### 用openssl签一个证书部署到apache或nginx
<br>

---
# 四、架构篇
---
## 分布式
<br>
### 数据一致性、服务治理、服务降级
<br>
### 分布式事务
2PC、3PC、CAP、BASE、 可靠消息最终一致性、最大努力通知、TCC
<br>
### Dubbo
服务注册、服务发现，服务治理
<br>
### 分布式数据库
怎样打造一个分布式数据库、什么时候需要分布式数据库、mycat、otter、HBase
<br>
### 分布式文件系统
mfs、fastdfs
<br>
### 分布式缓存
缓存一致性、缓存命中率、缓存冗余
<br>

---
## 微服务
<br>
### SOA、康威定律
<br>
### ServiceMesh
<br>
### Docker & Kubernets
<br>
### Spring Boot
<br>
### Spring Cloud
<br>

---
## 高并发
<br>
### 分库分表
<br>
### CDN技术
<br>
### 消息队列
ActiveMQ、RabbitMQ
<br>

---
## 监控
<br>
### 监控什么
CPU、内存、磁盘I/O、网络I/O等
<br>
### 监控手段
进程监控、语义监控、机器资源监控、数据波动
<br>
### 监控数据采集
日志、埋点
<br>
### Dapper
<br>

---
## 负载均衡
tomcat负载均衡、Nginx负载均衡
<br>

---
## DNS
DNS原理、DNS的设计
<br>

---
## CDN
数据一致性
<br>

---
# 五、 扩展篇
---
## 云计算
IaaS、SaaS、PaaS、虚拟化技术、openstack、Serverlsess
<br>

---
## 搜索引擎
Solr、Lucene、Nutch、Elasticsearch
<br>

---
## 权限管理
Shiro
<br>

---
## 区块链
哈希算法、Merkle树、公钥密码算法、共识算法、Raft协议、Paxos 算法与 Raft 算法、拜占庭问题与算法、消息认证码与数字签名
<br>
### 比特币
挖矿、共识机制、闪电网络、侧链、热点问题、分叉
<br>
### 以太坊
<br>
### 超级账本
<br>

---
## 人工智能
数学基础、机器学习、人工神经网络、深度学习、应用场景。
<br>
### 常用框架
TensorFlow、DeepLearning4J
<br>

---
## 其他语言
Groovy、Python、Go、NodeJs、Swift、Rust
<br>

---
# 六、 推荐书籍
---
《深入理解Java虚拟机》 
《Effective Java》 
《深入分析Java Web技术内幕》 
《大型网站技术架构》 
《代码整洁之道》 
《Head First设计模式》 
《maven实战》 
《区块链原理、设计与应用》 
《Java并发编程实战》 
《鸟哥的Linux私房菜》 
《从Paxos到Zookeeper》 
《架构即未来》
<br>

---