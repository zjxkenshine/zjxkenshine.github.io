---
title: Java学习过程中遇到的问题及解决方案
date: 2018-5-16 19:47:09  
tags: 
- Java
- JVM
categories: Java基础

---
## 0. 遇到的问题
1. JVM学习时使用JPS命令报错：
`Exception in thread "main" java.lang.NullPointerException`

---
## 1.问题1~5解决方案：
**1)JVM学习时使用JPS命令报错：**
1. 详细报错信息：
		Exception in thread "main" java.lang.NullPointerException
				at sun.jvmstat.perfdata.monitor.protocol.local.LocalVmManager.activeVms(
		LocalVmManager.java:148)
				at sun.jvmstat.perfdata.monitor.protocol.local.MonitoredHostProvider.act
		iveVms(MonitoredHostProvider.java:150)
				at sun.tools.jps.Jps.main(Jps.java:62)
2. 解决方法：
jp只能显示当前用户的java进程，JVM是管理员而我是使用当前用户的方式运行cmd,所以一直找不到文件，就报错了。
以管理员方式运行cmd就可以了：
![](http://p5ki4lhmo.bkt.clouddn.com/00063Java%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-01.jpg)


---