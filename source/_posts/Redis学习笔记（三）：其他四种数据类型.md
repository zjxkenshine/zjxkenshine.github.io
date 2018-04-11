---
title: Redis学习笔记（三）：其他四种数据类型的命令操作
date: 2018-04-11 10:44:27
tags: Redis
categories: NoSQL

---
## 0.学习准备
1. 参考资料：
	参考视频：《动力节点Redis视频教程》
	参考教程：<http://www.redis.net.cn/tutorial/3501.html>
	Redis命令参考手册：
	<http://redisdoc.com/>
2. 准备：
开启Redis服务并连接Redis客户端
String（字符串），Hash(哈希表)，List(列表)，Set(集合)，SortedSet（有序集合）
String的操作已经学习过了。

---
## 1.哈希hash类型的命令操作
**1)hash简介：**
1. hash结构
是一个String类型的file和value的映射表，特别适合用于存储对象。
存储结构示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-01.jpg)
2. 相当于是Java中的一个Map，里面存的是键值对的集合。

**2)常用的重要命令：**
1. `hset`：哈希表设置值的方式（一次只能存储一个字段field）
`hset key field value`
将哈希表key中的域field的值设为value。
如果key不存在，一个新的哈希表被创建并进行HSET操作。
如果域field已经存在于哈希表中，旧值将被覆盖。
返回值：
如果field是哈希表中的一个新建域，并且值设置成功，返回 1 。
如果哈希表中域field已经存在且旧值已被新值覆盖，返回 0 。
2. `hget`：获取hash表中的一个字段field的值
`hget key field`
返回哈希表key中给定域field的值。
不存在则返回nil。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-02.jpg)
3. `hmset`：同时设置多个域-值(field-value)到key中
`hmset key field1 value1 field2 value2...`
会覆盖哈希表中已存在的域。
如果key不存在，一个空哈希表被创建并执行 HMSET 操作。
返回值：
如果命令执行成功，返回 OK 。
当 key 不是哈希表(hash)类型时，返回一个错误。
4. `hmget`：获取hash表中的一个或者多个field的值。
`hmget key field1 field2...`
返回哈希表key中，一个或多个给定域的值。
如果给定的域不存在于哈希表，那么返回一个nil值。
因为不存在的key被当作一个空哈希表来处理，所以对一个不存在的key进行hmget操作将返回一个只带有nil值的表。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-03.jpg)
5. `hgetall key`：
返回哈希表key中，所有的域和值。
在返回值里，紧跟每个域(field)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。
6. `hdel`：删除一个或多个指定域
`hdel key field1 field2...`
删除哈希表key中的一个或多个指定域，不存在的域将被忽略。
返回值：
被成功移除的域的数量，不包括被忽略的域。
7. `hkeys`：返回哈希表key中的所有的域
`hkeys key`，注意只会有fields,不会显示域中的值
当key不存在时，返回一个空表。
8. `hvals`：返回哈希表key中的所有域的值
`hvals key`，不会返回域名。
返回值：
一个包含哈希表中所有值的表。
当key不存在时，返回一个空表。
测试后面几个命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-04.jpg)

**3)其他的常用命令：**
1. `hlen key`：获取hash表key中域field的个数
当key不存在时，返回 0
2. `hexists`：查看域filed是否存在
`hexists key field`
查看哈希表key中，给定域field是否存在。
3. `hsetnx`：设置值，不覆盖
`hsetnx key field value`
将哈希表key中的域field的值设置为value，当且仅当域field不存在。
若域field已经存在，该操作无效。
如果key不存在，一个新哈希表被创建并执行 HSETNX 命令。
注意，没有`hmsetnx`命令。
测试以上三个命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-05.jpg)
4. `hincrby`:增加步长
`hincrby key field increment`
为哈希表key中的域field的值加上增量increment。
增量也可以为负数，相当于对给定域进行减法操作。
如果域field不存在，那么在执行命令前，域的值被初始化为 0 。
对一个储存字符串值的域field执行 HINCRBY 命令将造成一个错误。
本操作的值被限制在64位(bit)有符号数字表示之内。
5. `hincrbyfloat`：增加浮点数增量
`hincrbyfloat key field increment`
为哈希表key中的域field加上浮点数增量increment。
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-06.jpg)

---
## 2.列表List类型的命令操作
**1)List类型简介：**
1. list结构：
key是string类型，value是简单的**字符串**列表，根据插入的顺序排序，可以插入一个数据到列表的头部(左)或者列表的尾部(右)。
2. 存储示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-07.jpg)

**2)常用的重要命令：**
1. `lpush`:插入到表头(左边)，可多个
`lpush key value1 value2 value3...`
将一个或多个值value插入到列表key的表头
如果有多个value值，那么各个value值按从左到右的顺序依次插入到表头
如有列表456，执行lpush key 123后的值是：321456，
相当于依次执行`lpush key 1`,`lpush key 2`和`lpush key 3`。
2. `rpush`:插入到表尾(右边)，可多个
`rpush key value1 value2 value3...`
将一个或多个值value插入到列表key的表尾
如果有多个value值，那么各个value值按从左到右的顺序依次插入到表尾
3. `lrange`：取列表中的元素
`lrange key start stop`
a. 返回列表key中指定区间内的元素，区间以偏移量start和stop指定。
	下标(index)参数start和stop都以0为底，也就是说以0表示列表的第一个元素，以1表示列表的第二个元素，以此类推。包括stop为下标的那个值。
	你也可以使用负数下标，以-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
b. 超出范围的下标值不会引起错误。
	如果start下标比列表的最大下标end(LLEN list减去1)还要大，那么LRANGE返回一个空列表。
	如果stop下标比end下标还要大，Redis将stop的值设置为end。
4. 测上面的三个方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-08.jpg)
取所有元素：`lrange key 0 -1`就可以了
5. `lpop key`移除并返回列表key的列表头元素。
删除并返回最左边的元素，不存在时返回nil
6. `rpop key`:移除并返回列表 key 的尾元素。
删除并返回最右边的元素，不存在时返回nil
7. `lindex `：取某一下标的值
`lindex key index`
返回列表key中,下标为index的元素。
下标(index)参数start和stop都以0为底。
你也可以使用负数下标，以-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
如果key不是列表类型，返回一个错误。
相当于使用`lrange key index index`。
8. `llen`：返回长度
`LLEN key`
返回列表key的长度。
如果key不存在，则key被解释为一个空列表，返回0.
如果key不是列表类型，返回一个错误。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-09.jpg)

**3)其它常用的操作命令：**
1. `lrem`：删除某一数量的特定值
`lrem key count val`
根据参数 count 的值，移除列表中与参数 value 相等的元素。
count 的值可以是以下几种：
count>0:从表头开始向表尾搜索，移除与value相等的元素，数量为count 。
count<0:从表尾开始向表头搜索，移除与value相等的元素，数量为count的绝对值。
count=0:移除表中所有与value相等的值。
删除成功则返回被删除的元素数量，删除失败则返回0。
2. `ltrim`：删除指定区域外的元素
`ltrim key start stop`
对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
举个例子，执行命令`LTRIM list 0 2`，表示只保留列表 list 的前三个元素，其余元素全部删除。下标(index)参数start和stop都以0为底。
也可以使用负数下标，以-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
测试前两个方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-10.jpg)
3. `rpoplpush`：rpop+lpush
`rpoplpush source destination`
将列表source中的最后一个元素(尾元素)弹出，并返回给客户端。
将source弹出的元素插入到列表destination，作为destination列表的的头元素。
如果source不存在，值nil被返回，并且不执行其他动作。
如果source和destination相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。
4. `lset`：替换
`lset key index value`
将列表key下标为index的元素的值设置为value。
当index参数超出范围，或对一个空列表(key不存在)进行LSET时，返回一个错误。
5. `linsert` 
`linsert key BEFORE|AFTER pivot value`
将值value插入到列表key当中，位于值pivot之前(before)或之后(after)。
当pivot不存在于列表key时，不执行任何操作。
当key不存在时，key被视为空列表，不执行任何操作。
如果key不是列表类型，返回一个错误。
注意pivot和value的值需要用双引号连接。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-11.jpg)

**4)三个阻塞式命令：**
1. `blpop`：列表的阻塞式(blocking)左弹出原语
`blpop key [key ...] timeout`
	- 它是LPOP命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被BLPOP命令阻塞，直到等待超时或发现可弹出元素为止。
	- 当给定多个key参数时，按参数key的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。
	- 如果所有给定key都不存在或包含空列表，那么 BLPOP 命令将阻塞连接，直到等待超时，或有另一个客户端对给定key的任意一个执行LPUSH或RPUSH命令为止。
	- 超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为0表示阻塞时间可以无限期延长(block indefinitely) 。
2. `brpop`：列表的阻塞式右弹出原语
`brpop key [key ...] timeout`
和blpop相同。
3. `brpoplpush`：
`brpoplpush source destination timeout`
BRPOPLPUSH是RPOPLPUSH的阻塞版本，当给定列表source不为空时，BRPOPLPUSH的表现和RPOPLPUSH一样。
当列表source为空时，BRPOPLPUSH命令将阻塞连接，直到等待超时，或有另一个客户端对source执行 LPUSH 或 RPUSH 命令为止。
超时参数timeout接受一个以秒为单位的数字作为值。超时参数设为0表示阻塞时间可以无限期延长。

---
## 3.集合set类型的命令操作
**1)Set类型简介：**
1. set和list相似，set是String类型的**无序集合**，里面的值不能被重复，相同的值只存储一个。
2. 和Java中的集合差不多。
结构示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-12.jpg)

**2)常用的重要命令：**
1. `sadd`：添加数据（元素）
`sadd key member1 member2...`
将一个或多个member元素加入到集合key当中，已经存在于集合的member元素将被忽略。假如key不存在，则创建一个只包含member元素作成员的集合。
当key不是集合类型时，返回一个错误。
2. `smembers key`：返回集合key中的所有成员。
不存在的key被视为空集合。
3. `sismember`：判断是否在集合内。
`sismember key member`
判断member元素是否集合key的成员。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-13.jpg)
4. `scard key`：返回集合key中的元素个数
5. `srem`：删除元素
`srem key member1 member2...`
移除集合key中的一个或多个member元素，不存在的member元素会被忽略。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-14.jpg)
6. `srandmember`：随机返回一个元素
`srandmember key [count]`
如果命令执行时，只提供了key参数，那么返回集合中的一个随机元素。不会改变集合内的元素。
支持一个count参数：
	- 如果count为正数，且小于集合基数，那么命令返回一个包含count个元素的数组，数组中的元素各不相同。如果count大于等于集合基数，那么返回整个集合。
	- 如果count为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为count的绝对值。
7. `spop key`：移除并返回key中随机一个元素
如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用SRANDMEMBER命令。
8. `smove`：移动元素
`smove source destination member`
将member元素从source集合移动到destination集合。
smove是原子性操作。
如果source集合不存在或不包含指定的member元素，则SMOVE命令不执行任何操作，仅返回0。否则，member元素从source集合中被移除，并添加到destination集合中去。
当destination集合已经包含member元素时，SMOVE命令只是简单地将source集合中的member元素删除。
当source或destination不是集合类型时，返回一个错误。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-15.jpg)
更多命令查看命令参考手册。

---
## 4.有序集合SortedSet(ZSet)类型的命令操作
**1)ZSet简介**
1. 有序集合ZSet：
和集合set一样是String类型的集合，且不允许重复的成员。
不同的是有序集合的每一个元素都会关联一个分数(分数可以重复)，redis通过分数来为集合进行从小到大的排序。
所以很多的操作都和Set的操作类似。
2. 结构示意图如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-16.jpg)

**2)常用的重要命令：**
1. `zadd`：添加成员及分数
`zadd key score member [[score member] [score member] ...]`
将一个或多个member元素及其score值加入到有序集key当中。
如果某个member已经是有序集的成员，那么更新这个member的score值，并通过重新插入这个member元素，来保证该member在正确的位置上。
score值可以是整数值或双精度浮点数。
如果key不存在，则创建一个空的有序集并执行`ZADD`操作。
当key存在但不是有序集类型时，返回一个错误。
2. `zrem`：删除有序集合中一个或者多个成员及其分数
`zrem key member1 member2...`
当key存在但不是有序集类型时，返回一个错误。
3. `zrange`：获取指定区间的元素
`zrange key start stop [WITHSCORES]`
	- 返回有序集 key 中，指定区间内的成员。
	其中成员的位置按score值递增(从小到大)来排序。
	具有相同score值的成员按字典序(lexicographical order)来排列。
	- 如果你需要成员按score值递减(从大到小)来排列，请使用 `ZREVRANGE`命令。
	- 下标参数start和stop都以 0 为底。
	也可以使用负数下标，以-1表示最后一个成员，-2表示倒数第二个成员，以此类推。
	超出范围的下标并不会引起错误。
	- 可以通过使用WITHSCORES选项，来让成员和它的score值一并返回，返回列表以 `value1,score1, ..., valueN,scoreN`的格式表示。客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。
4. `zrevrange`：和上面差不多，只不过是从大到小排的。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-17.jpg)
5. `zcard key`：返回key中元素的个数
6. `zrank`：查找某一成员所在排名，递增
`zrank key member`
返回有序集key中成员member的排名。其中有序集成员按score值递增(从小到大)顺序排列。
排名以0为底，也就是说，score值最小的成员排名为0。
使用`ZREVRANK`命令可以获得成员按score值递减(从大到小)排列的排名。
7. `zrevrank`：查找某一成员所在排名，递减
`zrevrank key member`，其他注意事项同上。
8. `zscore`：获取某一成员的分数值
`zscore key member`
如果member元素不是有序集key的成员，或key不存在，返回nil。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00035Redis%E5%AD%A6%E4%B9%A03-18.jpg)

**3)其他常用命令：**
1. `ZRANGEBYSCORE`：score定区间查找（顺序）
`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
	- 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。
	- 可选的 LIMIT 参数指定返回结果的数量及区间(就像SQL中的 SELECT LIMIT offset, count )，注意当 offset 很大时，定位 offset 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。
	- 可选的 WITHSCORES 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 score 值一起返回。
	- 区间及无限:
	min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 ZRANGEBYSCORE 这类命令。
	默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。
2. `ZREVRANGEBYSCORE`：score定区间查找（逆序）
语法及注意事项和上一个命令一样。
3. `zcount`：返回区间中的成员数量
`ZCOUNT key min max`
返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员的数量。
min与max的使用方法参照`ZRANGEBYSCORE`函数。
4. 还有其他一些不太常用的命令可以参考命令手册：
<http://redisdoc.com/>

---
## 5.重要命令总结(更多查看命令手册)
**1)关于Hash的命令：**
`hset key field value`： 将哈希表key中的域field的值设为value（仅能存一个）
`hget key field`： 返回哈希表key中给定域field的值。
`hmset key field1 value1 field2 value2...`： 同时设置多个域-值(field-value)到key中
`hmget key field1 field2...`： 返回哈希表key中，一个或多个给定域的值。
`hgetall key`： 返回哈希表key中，所有的域和值。
`hdel key field1 field2...`： 删除哈希表key中的一个或多个指定域，不存在的域将被忽略。
`hkeys key`： 返回哈希表key中的所有的域
`hvals key`： 返回哈希表key中的所有域的值

`hlen key`： 获取hash表key中域field的个数
`hexists key field`： 查看哈希表key中，给定域field是否存在。
`hsetnx key field value`： 将哈希表key中的域field的值设置为value，不重复时才成功。
`hincrby key field increment`： 为哈希表key中的域field的值加上增量increment。increment为负则是减少。
`hincrbyfloat key field increment`： 为哈希表key中的域field加上浮点数增量increment。

**2)关于List的命令：**
`lpush key value1 value2 value3...`： 将一个或多个值value插入到列表key的表头（左边）
`rpush key value1 value2 value3...`： 将一个或多个值value插入到列表key的表尾(右边)
`lrange key start stop`： 返回列表key中指定区间内的元素，区间以偏移量start和stop指定。
获取所有元素： `lrange key 0 -1`
`lpop key`： 移除并返回列表key的表头（最左边）元素。
`rpop key`： 移除并返回列表key的尾元素（最右边）。
`lindex key index`： 返回列表key中,下标为index的元素。
`llen key`： 返回列表key的长度。

`lrem key count val`： 根据参数count的值，移除列表中与参数value相等的元素。count为0则移除全部。
`ltrim key start stop`： 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
`rpoplpush`：rpop+lpush
`rpoplpush source destination`： 将列表source中的最后一个元素(尾元素)弹出，并作为destination列表的的头元素插入。
`lset key index value`： 将列表key下标为index的元素的值设置为value。
`linsert key BEFORE|AFTER pivot value`： 将值value插入到列表key当中，位于值pivot之前(before)或之后(after)。

`blpop key [key ...] timeout`： 列表的阻塞式(blocking)左弹出原语
`brpop key [key ...] timeout`： 列表的阻塞式右弹出原语

`brpoplpush source destination timeout`：BRPOPLPUSH是RPOPLPUSH的阻塞版本，当给定列表source不为空时，BRPOPLPUSH的表现和RPOPLPUSH一样。
当列表source为空时，BRPOPLPUSH命令将阻塞连接，直到等待超时，或有另一个客户端对source执行LPUSH或RPUSH命令为止。

**3)关于Set的命令：**
`sadd key member1 member2...`： 将一个或多个member元素加入到集合key当中
`smembers key`： 返回集合key中的所有成员。
`sismember key member`： 判断member元素是否集合key的成员。
`scard key`： 返回集合key中的元素个数
`srem key member1 member2...`： 移除集合key中的一个或多个member元素，不存在的member元素会被忽略。
`srandmember key [count]`： 随机返回一个(或count个)元素，不删除集合内容
`spop key`：移除并返回key中随机一个元素
`smove source destination member`： 将member元素从source集合移动到destination集合。

**4)关于ZSet的操作：**
`zadd key score member [[score member] [score member] ...]`： 将一个或多个member元素及其score值加入到有序集key当中。
`zrem key member1 member2...`： 删除有序集合中一个或者多个成员及其分数
`zrange key start stop [WITHSCORES]`： 返回有序集key中，指定区间内的成员。分数从小到大排。
`zrevrange key start stop [WITHSCORES]`： 返回有序集key中，指定区间内的成员。分数从大到小排。
`zcard key`：返回key中元素的个数
`zrank key member`： 返回有序集key中成员member的排名。其中有序集成员按score值递增(从小到大)顺序
`zrevrank key member`： 查找某一成员所在排名，递减。
`zscore key member`： 获取某一成员的分数值，如果member元素不是有序集key的成员，或key不存在，返回nil。

`ZRANGEBYSCORE`： score定区间查找（顺序）
`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`： 返回有序集key中，所有score值介于min和max之间(包括等于min或max)的成员。有序集成员按score值递增(从小到大)次序排列。
`ZREVRANGEBYSCORE`： score定区间查找（逆序）
`ZCOUNT key min max`： 返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员的数量。


---