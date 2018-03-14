---
title: Centos7关于几个常见问题的解决方案
date: 2018-03-10 10:24:58
tags: Linux
categories: 操作系统

---
## 0.我遇到的问题列表（关于用户/管理员的）
1. Centos7忘记密码怎么解决?



---
## 1.Centos7忘记密码怎么解决
太久没使用Linux（Centos7）了，忘记root密码了怎么办？
1. 在开机过程中，快速按下键盘上的方向键↑和↓。目的是告知引导程序，我们需要在引导页面选择不同的操作，以便让引导程序暂停。 
![](http://p5ki4lhmo.bkt.clouddn.com/00006centos%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%981.jpg)
然后选择忘记密码的那个系统按下`e`键进入编辑模式。
2. 进入之后，将光标一直移动到 LANG=en_US.UTF-8 后面，空格，再追加init=/bin/sh。这里特别注意，需要写在UTF-8后，保持在同一行，并注意空格。由于屏幕太小，会自动添加\换行，这个是正常的。（移动光标时上下换段左右换段内文字）
![](http://p5ki4lhmo.bkt.clouddn.com/00006centos%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%982.jpg)
3. 按下CTRL+X进行引导启动，成功后进入该界面
![](http://p5ki4lhmo.bkt.clouddn.com/00006centos%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%983.jpg)
有时可能会产生这样的错误
![](http://p5ki4lhmo.bkt.clouddn.com/00006centos%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%984.jpg)
可以查看以下第2步是否写对（注意=后面有斜杠）
4. 输入以下命令：
挂载根目录 
		mount -o remount, rw /
选择要修改密码的用户名，这里选择root用户进行修改，可以更换为你要修改的用户
		passwd root
输入2次一样的新密码，注意输入密码的时候屏幕上不会有字符出现。 
如果输入的密码太简单，会提示警告（BAD PASSWORD：The password fails the dictionary check - it is too simplistic/systematic），可以无视它，继续输入密码，不过建议还是设置比较复杂一些的密码，以保证安全性，如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00006centos%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%985.jpg)
6. 如果已经开启了SElinux，则需要输入以下命令 
		touch /.autorelabel
7. 最后输入以下命令重启系统即可：
		exec /sbin/init 
或
		exec /sbin/reboot
6. 转自:
<http://blog.csdn.net/wcy00q/article/details/70570043>

---

