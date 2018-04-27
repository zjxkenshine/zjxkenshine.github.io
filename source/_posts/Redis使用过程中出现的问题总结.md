---
title: Redis使用过程中出现的问题及解决方案
date: 2018-4-25 11:51:56  
tags: Redis
categories: NoSQL

---
## 0.问题列表
1. 安装ruby-redis接口时
`# gem install redis --version 4.0.0`
出现问题：
		ERROR:  Loading command: install (LoadError)  
			cannot load such file -- zlib  
		ERROR:  While executing gem ... (NoMethodError)  
		undefined method `invoke_with_build_args' for nil:NilClass 
2. 使用`gem sources -a https://ruby.taobao.org/`切换安装源时出错
		ERROR:  While executing gem ... (Gem::Exception)  
		Unable to require openssl, install OpenSSL and rebuild ruby (preferred) or use non-HTTPS sources
3. 使用redis-trib.rb进行节点扩容(移动槽)时出现问题：(待解决)
		[ERR] Calling MIGRATE: ERR Syntax error, try CLIENT (LIST | KILL | GETNAME | SETNAME | PAUSE | REPLY)
再次启动出现如下问题：
		[WARNING] Node 127.0.0.1:8000 has slots in migrating state (206).
		[WARNING] Node 127.0.0.1:8006 has slots in importing state (206).
		[WARNING] The following slots are open: 206


---
## 1.问题1-5解决方案
**1)安装ruby-redis接口时出错：**
1. 使用命令：
`# gem install redis --version 4.0.0`
问题描述：
		ERROR:  Loading command: install (LoadError)  
			cannot load such file -- zlib  
		ERROR:  While executing gem ... (NoMethodError)  
		undefined method `invoke_with_build_args' for nil:NilClass
2. 原因缺少zlib依赖，需要安装zlib库
3. 第一步：安装zlib库，如果已经安装，跳过：
`# yum install zlib-devel`
第二步：集成zlib到ruby
		cd /usr/local/ruby-2.5.1/ext/zlib
		ruby extconf.rb
		vim Makefile
		//在操作下一步之前需要修改Makefile文件中的zlib.o: $(top_srcdir)/include/ruby.h,将$(top_srcdir)修改为../..如下
		//zlib.o: ../../include/ruby.h
		//这一步如果不修改，make时会爆出另外一个错误
		//make:*** No rule to make target `/include/ruby.h', needed by `zlib.o'.  Stop
		make && make install
4. 修改的那里在Makefile文件的最底部，如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00050Redis%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-01.jpg)

**2）使用`gem sources -a`切换安装源时出错：**
1. 使用命令：
`gem sources -a https://ruby.taobao.org/`
错误描述：
		ERROR:  While executing gem ... (Gem::Exception)
		Unable to require openssl, install OpenSSL and rebuild ruby (preferred) or use non-HTTPS sources
2. 错误原因：
缺少openssl库
3. 安装openssl库：
		# yum install openssl-devel
不要只用yum install openssl来安装，否则会缺少pcre等相关库，执行ruby extconf.rb会提示找不到ssl.h文件。
4. 集成到ruby：
		cd /usr/local/ruby-2.5.1/ext/openssl
		ruby extconf.rb
		vim Makefile
		//同样修改Makefile中的$(top_srcdir)为../..
		make && make install
5. 大概是在Makefiles所有`$(top_srcdir)`修改为`../..`
但是有点多，所以在最顶端加上一句
`top_srcdir=../..`
就可以了，不用去手动修改$(top_srcdir)。

**3)使用redis-trib.rb进行节点扩容(移动槽)时出现问题**
1. 出错问题：
		[ERR] Calling MIGRATE: ERR Syntax error, try CLIENT (LIST | KILL | GETNAME | SETNAME | PAUSE | REPLY)
		*** Please fix your cluster problems before resharding
再次启动出现如下问题：
		>>> Check for open slots...
		[WARNING] Node 127.0.0.1:8000 has slots in migrating state (206).
		[WARNING] Node 127.0.0.1:8006 has slots in importing state (206).
		[WARNING] The following slots are open: 206
		>>> Check slots coverage...
2. 查阅资料发现可能是
某个很大的key堵塞了redis导致，但是并不是：
		127.0.0.1:8000> cluster getkeysinslot 206 100
		1) "jinan"
		127.0.0.1:8000> debug object jinan
		Value at:0x7efbf0821e20 refcount:1 encoding:embstr serializedlength:4 lru:14855605 lru_seconds_idle:7818
参考地址：
<https://www.cnblogs.com/svan/p/6039942.html>
3. 发现可以使用`redis-trib.rb fix`来修复节点：
		# redis-trib.rb fix 127.0.0.1:8000
但是无法修复：
		>>> Fixing open slot 206
		Set as migrating in: 127.0.0.1:8000
		Set as importing in: 127.0.0.1:8006
		Moving slot 206 from 127.0.0.1:8000 to 127.0.0.1:8006: 
		[ERR] Calling MIGRATE: ERR Syntax error, try CLIENT (LIST | KILL | GETNAME | SETNAME | PAUSE | REPLY)
参考地址：
<http://xiaorui.cc/2015/05/19/%E4%BD%BF%E7%94%A8redis-trib-fix%E5%91%BD%E4%BB%A4%E4%BF%AE%E5%A4%8Dredis-cluster%E8%8A%82%E7%82%B9/>
4. 仔细看了看那个错误的意思：
尝试在客户端杀掉这个卡住的客户端，进入8000客户端，使用client list查看客户端，并把时间最久的那个客户端kill掉：
![](http://p5ki4lhmo.bkt.clouddn.com/00050Redis%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-02.jpg)
也没有用。杀不掉。
5. 尝试手动去迁移槽中的key(只有一个jinan啊，什么情况啊)
使用cluster migrate也失败了。。。放弃了。。。
这个问题等以后工作后遇到运维大佬再解决了。
6. 现在学习阶段的解决方法就是：
		# FLUSHALL  //配置文件中未关闭
		# redis-trib.rb fix 127.0.0.1:8000
然后再次进行分配，发现槽点不均匀的话，使用
		# redis-trib.rb rebalance 127.0.0.1:8000
就可以分配均匀了,可以使用下命令查看：
		# redis-trib.rb check 127.0.0.1:8000

---