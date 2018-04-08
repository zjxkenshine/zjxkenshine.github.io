---
title: Windows命令行cmd中遇到的问题及解决
date: 2018-4-8 18:48:03 
tags: Windows
categories: 操作系统

---
## 0. 遇到的问题
1. `ping`不是内部或者外部命令
2. `telnet`不是内部或者外部命令

---
## 1.问题1-5的解决
**1)ping不是内部或者外部命令**
很多时候需要用ping来测试windows与Linux是否连接成功。
出现`ping`不是内部或者外部命令这种情况只需要：
打开我的电脑-->系统属性-->高级系统设置-->环境变量-->系统变量-->PATH
查看是否有`C:\windows\system32`的配置，没有则加上;然后添加该配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-01.jpg)
再ping一下虚拟机
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-02.jpg)
0%丢失就说明连接成功了。

**2)`telnet`不是内部或者外部命令**
1. 在使用Redis时会用telnet测试redis的端口是否开启
然后报telnet`不是内部或者外部命令错误。
2. 点击【计算机】-->【控制面板】-->【程序】
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-03.jpg)
然后点击【程序和功能】
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-04.jpg)
点击左侧【启动或关闭Windows功能】
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-05.jpg)
勾选Telnet客户端：
![](http://p5ki4lhmo.bkt.clouddn.com/00032cmd%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%881-06.jpg)
点击确定，等待几分钟安装完毕就可以了。


---