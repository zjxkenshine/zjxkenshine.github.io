---
title: Redis学习笔记（八）：Redis Cluster集群简单搭建
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


---