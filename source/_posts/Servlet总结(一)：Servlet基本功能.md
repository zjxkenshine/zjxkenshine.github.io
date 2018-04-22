---
title: Servlet总结(一)：Servlet基本功能
date: 2018-4-20 13:59:49  
tags: Servlet
categories: JavaWeb

---
## 0.参考资料
各类博客，资料，课本等
Servlet,Jsp这种东西已经用烂了，所以就不测试了。
但是很多理论的东西还很薄弱，所以就只记载一些知识点。

---
## 1.Servlet简介
1. 什么是Servlet:简单概念
servlet是在服务器上运行的小程序。Servlet的主要功能在于交互式地浏览和修改数据，生成动态Web内容，是为web开发服务的。
2. Servlet的详细介绍：
我们在网上浏览网页，需要一个web服务器，浏览网页的过程就是浏览器通过HTTP协议与WEB服务器 交互的过程。在过去，大多是静态网页，因此只须把资源放在WEB服务器上即可。如今随着应用的发展，客户与服务器需要动态的交互，为了实现这一目标，就需 要开发一个遵循HTTP协议的服务器端应用软件，来处理各种请求。那么servlet是一个基于java技术的WEB组件，运行在服务器端，我们利用 sevlet可以很轻松的扩展WEB服务器的功能，使它满足特定的应用需要。
3. 关于Servlet容器：
servlet由servlet容器管理，servlet容器也叫 servlet引擎，是servlet的运行环境，给发送的请求和响应之上提供网络服务。比如tomcat就是我们常用的一个servlet容器，其接受 客户端并做出响应的步骤如下：
	>1、客户端访问WEB服务器，发送HTTP求
	2、WEB服务器接收到请求后，传递给servlet容器
	3、servlet容器加载servlet，产生servlet实例，并向其传递表示请求和响应的对响
	4、servlet得到客户端的请求信息，并进行相应的处理  
	5、servlet实例把处理结果发送回客户端，容器负责确保响应正确送出，同时将控制返回给WEB服务器
4. 上述运行过程示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00045Servlet%E6%80%BB%E7%BB%931-01.jpg)

---
## 2.Servlet类结构


---
## 3.Servlet生命周期

---

