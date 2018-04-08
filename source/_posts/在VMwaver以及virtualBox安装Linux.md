---
title: 在VMwaver以及VirtualBox安装Linux
date: 2018-04-07 16:10:07
tags: Linux
categories: 操作系统

---
## 0.前言
本来是一直都在用VMwaver装的Centos7.0,但是最近学Redis需要用Xftp传文件，又需要用桥接，校园网的桥接无法联网。NAT模式下虽然可以用端口转发连接，但是不知道是什么问题总会断开连接。(对于一个新手而言弄了一天没弄好已经快奔溃了)
所以换个虚拟机试试，顺便把VMwaver的磁盘内存改小一点，原来分配了30G感觉太大了，设置小一点。

---
## 1.下载Centos镜像
1. 下载地址
下载地址大全：
<http://mirror.centos.org/centos/7/isos/>
阿里云地址：
<https://mirrors.aliyun.com/centos/>
2. 以阿里云为例，选择需要的版本点击iso再点击64位，之后进入这个页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux01.jpg)
选择需要的版本，点击.iso文件就可以下载了。
我以前下载的是Everything版本，现在下载的是DVD版本。

---
## 2.在VMwaver安装及配置Linux
VMwaver版本12.0。
### 导入iso镜像文件
1. 点击【文件】-->【新建虚拟机】
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux02.jpg)
就使用默认的典型推荐，下一步。
2. 选择第二个，最简单的方式，毕竟我是新手，怎么简单怎么来：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux03.jpg)
下一步会让你设置账号密码用于登录，设置好就行，再点击下一步。
3. 接着设置虚拟机名称及存储地址：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux04.jpg)
下一步。
4. 选择磁盘大小：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux05.jpg)
这里是不是存储为单个文件对于我而言没什么关系，上一次是存储为单个文件，那这次就选择多个吧。下一步。
5. 选择自定义设置将一些不需要的设备移除（如打印机）：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux06.jpg)
点击完成，这样iso镜像文件就导入完毕了。下面还需要配置Centos7

### 配置Centos7
1. 点击开启此虚拟机，运行虚拟机，可以看到如下界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux07.jpg)
2. 选择DATE&TIME,将时间改为东八区并勾选右上角使用网络时间:
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux08.jpg)
点击左上角的done确认并返回。
3. LANGUAGE SUPPORT选择中文支持：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux09.jpg)
-->Done
4. 添加一个中文键盘：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux10.jpg)
-->Done
返回后点击最右下角的BEGIN INSTALL开始安装。
5. 然后会到达这个页面：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux11.jpg)
点解USER CREATION创建用户：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux12.jpg)
创建完毕后选等待安装完毕就可以使用了。(可能需要等待一段时间)
6. 进去之后选择语言就可以了：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux12-01.jpg)

---
## 3.在VirtualBox安装及配置Linux
软件：VirtualBox5.28
### 新建虚拟机
1. 打开软件，新建-->新建虚拟机
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux13.jpg)
选择Linux,Red Hat或者Other Linux.点击下一步。
2. 设置内存大小为默认值：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux14.jpg)
下一步。
3. 创建虚拟硬盘：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux15.jpg)
4. 选择动态分配：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux16.jpg)
5. 设置虚拟硬盘大小：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux17.jpg)
6. 选择要保存的文件夹：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux17-01.jpg)
最后点击创建就创建完毕了。

### 设置虚拟机：
1. 点击VirtualBox最上方的设置(齿轮)进行配置：
2. 系统设置如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux18.jpg)
3. 在存储中点击分配光驱的右侧光盘，选择下载的iso文件导入。
注意选择第一控制器主通道。
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux19-1.jpg)
选择完毕是这个样子的：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux19-2.jpg)
4. 网络设置为NAT:
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux20.jpg)

### 安装Linux
1. 点击启动虚拟机，选择第一项(Install Centos7)：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux21-1.jpg)
按Enter回车键继续。
2. 安装环境选择中文：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux22.jpg)
点击继续。
3. 进去之后的界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux22-02.jpg)
注意有叹号的必须全部修改，否则无法继续。
4. 点击语言支持，默认是中文，选择额外一项英文：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux28.jpg)
点击左上角完成退出。
5. 点击键盘，选择额外的英文键盘支持：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux29.jpg)
点击左上角完成退出
6. 点击软件选择：
选择KDE界面，并勾选右侧的软件。
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux23-1.jpg)
兼容性程序因为只有一个版本所以没有选。
7. 选择安装位置：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux24.jpg)
8. 点击网络和主机名
配置以太网连接
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux30.jpg)
9. 然后返回来点击日期和时间，查看是否有左上角的网络时间，以及是否为自己所在的时区。
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux31.jpg)
10. 全部设置完毕是这个样子的：
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux32.jpg)
然后点击开始安装。
6. 需要设置root密码和创建用户
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux25.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux26.jpg)
设置完毕就等待它安装。
安装完毕后重启。
7. 重启后同意许可协议。
![](http://p5ki4lhmo.bkt.clouddn.com/00030%E5%AE%89%E8%A3%85Linux27.jpg)
然后就可以开始使用了。
但是可能是DVD版本的问题，键盘是不能输中文的。


---