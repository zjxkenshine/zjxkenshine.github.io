---
title: Centos7关于几个常见问题的解决方案
date: 2018-03-10 10:24:58
tags: Linux
categories: 操作系统

---
## 0.我遇到的问题列表（关于用户/管理员的）
1. Centos7忘记密码怎么解决?
2. 没有dump和restore命令怎么办?
3. vim打开了一个文件后还想打开另一个文件怎么办?



---
## 1.问题1~5解决方案
**1)Centos7忘记密码怎么解决?**
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

**2)Centos7没有dump与restore命令怎么办?**
既然没有装了，那就是默认不推荐使用了，备份可以cp或打包压缩嘛。备份文件系统可以用xfsdump。
如果执意要安装可以这么做:
输入以下命令：

		yum –y install dump*    //装不了就加*号，一般不加
结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-10.jpg)
再man dump一下：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-09.jpg)
可以用了。连restore也给你附带装好了。

**3)vim打开了一个文件后还想打开另一个文件怎么办?**
1. 原窗口打开新文件
		:open filename
但是这样打开文件无法使用`:n`、`:N`选择文件。
需要使用这些命令切换：
		[Ctrl]+6 ：互相切换
		:bn/N ：编辑下一个文件，最后一个文件的下一个为第一个文件
		:bp ：编辑上一个文件，第一个文件的上一个为最后一个
对于用(v)split在多个窗格中打开的文件，这种方法只会在当前窗格中切换不同的文件。
2. 在多个窗口打开：
		:vs/vsp ：文件路径/文件名      在新的垂直分屏中打开文件
		:sp ：文件路径/文件名      在新的水平分屏中打开文件
打开了多个窗口间切换：
		Ctrl+w+方向键——切换到前／下／上／后一个窗格
		Ctrl+w+h/j/k/l ——同上
		Ctrl+ww——依次向后切换到下一个窗格中
3. 多个个文件之间复制：
	- 在第一个文件中使用可视模式，就是VISUAL，然后选中要复制的文本，执行命令 `+y`,或者`*y`这就把内容复制到剪贴板。这里可以三个字符，而且一定要在可视化模式中，并存选中你要复制的代码以后，输入上面的命令，这时在VIM中的下面并不显示你输入的这条命令。
	- 在另一个文件中，执行命令`+p`,或者"`*p`。就能复制过来，＋指的是寄存器的意思,似乎也是操作系统的剪贴板，复制了之后，在别的地方，例如文本文件里就可以用ctrl＋v了。这里也是在可视模式下，不需要输入冒号`：`，这里输入的命令也是看不到的
4. 参考博客：
<http://blog.csdn.net/orangleliu/article/details/41745975>

---
