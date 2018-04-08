---
title: Xshell,Xftp的下载安装使用及使用过程中的问题
date: 2018-04-06 13:54:08
tags: Linux
categories: 操作系统

---
## 0.学习之前
本来想学习xmanager的，也不知道是以前玩的时候忘记破解了还是更新了破解失效了，反正显示评估过期了，网上也找不到xmanager评估过期的处理方式，只能学习xshell5和xftp5了：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp01.jpg)

---
## 1.下载安装
1. 下载网址：
<https://www.netsarang.com/download/software.html>
进去之后下拉可以看到这个下载页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp02.jpg)
最下面的一行显示Xshell和Xftp是有免费版的，点击这个`Free for Home&School`或者点击上面的Xshell5,Xftp5都可以到达免费下载页面。
当然，如果有钱或者有产品码也可以点击其他三个。
2. 以Xshell5为例，进去之后并不是让你直接下载，而是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp03.jpg)
选择Home&School user这一项，配置完带\*号的必填项并提交：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp04.jpg)
然后会给你的邮箱发一个邮件进行下载。
3. 登录邮箱，查看邮件：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp05.jpg)
然后点击他给的地址就可以直接下载**免费版**。
4. 安装：
点击下载的.exe文件按提示安装即可。

---
## 2.SSH的安装及启动
**1)SSH简介：**
1. 在使用Xshell和Xftp之前得先确保安装了SSH服务。
2. SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH最初是UNIX系统上的一个程序，后来又迅速扩展到其他操作平台。SSH在正确使用时可弥补网络中的漏洞。SSH客户端适用于多种平台。几乎所有UNIX平台—包括HP-UX、Linux、AIX、Solaris、Digital UNIX、Irix，以及其他平台，都可运行SSH。

**2)下载安装及启动SSH:**
1. 查看ssh服务的状态
root用户输入以下命令：
`service sshd status`
如果出现
`Loaded: error (Reason: No such file or directory)`
提示的话，说明没有安装ssh服务。
如果出现
`Active: inactive (dead)`
说明已经安装了ssh服务，但是没有开启。按照第三步：开启ssh服务。
2. 安装ssh服务
安装ssh命令：
centos7的root用户使用命令：
`yum install sshd`
或者
`yum install openssh-server`
然后按照提示，安装就好了。
3. 开启ssh服务(root用户)
在终端敲入以下命令：
`service sshd start`
执行完命令后，用第一步：查看ssh服务状态的命令，如果出现以下提示
`Active: active (running) since Sun 2013-04-07 13:43:11 CST; 15s ago`
重启：
`service sshd restart`
停止：
`service sshd stop`

4. 使用ssh服务
使用ssh服务跟使用ftp服务一样，推荐安装putty（一款远程登陆工具）来登陆本地主机。安装命令与第二步：安装ssh服务相同，只是把sshd换成putty即可。
安装putty完成后，使用以下命令远程登陆：
`putty ip/hostname`
其中ip/hostname为你的ssh主机的ip地址或者主机名
5. 卸载ssh服务
centos使用以下命令：
`yum remove sshd`
6. 我的ssh服务：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp09.jpg)

---
## 3.简单使用Xshell连接Linux
**1)Xshell简介：**
Xshell是一款常用的连接ssh服务器的软件，它通常用于远程登陆服务器，管理服务器等。最常用的就是在Windows下用它来连接Linux了。

**2)连接前提：**
1. 必须能联网。
2. 最好是使用桥接模式。
我的Linux是NAT模式联网的，而且是动态获取IP的，所以在NAT模式下的需要额外配置(如果虚拟机IP改变就要重新配置)。
3. 配置NAT模式支持xshell连接：
查看虚拟机的ip：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp07.jpg)
进行虚拟网络编辑器的设置：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp08.jpg)
4. NAT模式在校园网上下会有各种各样的问题(作为一个菜鸟真的炸了，以后再更NAT模式吧)，我换成了VirtualBox安装虚拟机，使用桥接进行连接。

**3)连接步骤：**
1. 首先是我的软件版本：
VirtualBox5.2.8，Centos7.4，Xshell5
2. 打开Xshell客户端，看到，会弹出一个会话，点击新建，如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp06.jpg)
3. 然后是virtualBox的网络设置：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp07-1.jpg)
4. 登录虚拟机，使用ifconfig查看ip:
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp08-1.jpg)
5. 然后在xshell添加配置，端口号为22，并设置账号密码：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp09-1.jpg)
可以先ping一下这个ip看是否可以连接再进行添加（另一个ip的图）:
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp15.jpg)
6. 登录成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp10.jpg)

---
## 4.Xftp连接Linux及简单使用
**1)连接Linux**
1. 方式一：
打开Xftp使用和Xshell一样的方式连接.
方式二：
在Xshell连接上之后点击新建文件传输：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp11.jpg)
2. 连接时出现的乱码问题：
在Xftp中：
点击【属性(上面的齿轮)】-->【选项】-->勾选UTF-8即可
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp12.jpg)

**2)简单使用**
1. 打开之后的界面如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp13.jpg)
2. 选择需要存放的位置以及传输给Linux或者Windows的文件点击传输就可以了
![](http://p5ki4lhmo.bkt.clouddn.com/00029XshellXftp14.jpg)

---
