---
title: 《鸟哥的Linux私房菜》笔记（二）初识命令行与正常关机
date: 2018-03-09 21:02:52
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第五章:首次登陆与在线求助man page

2. 本节重点：
- 命令行与图形界面的切换
- 简单命令的执行
- man page
- 常用热键
- 正确关机

---
## 1.首次登录系统
1. 为什么要登录:
Linux系统中由于是多人多任务的环境，所以系统随时都有很多任务在进行，因此正确关机很重要。不正常关机会导致文件系统错乱，造成数据的损毁，这也是为什么**通常Linux主机都会加挂一个不断电系统**。
2. Linux常见的图形化界面(X Window):GNOME和KDE
重启X Window快捷键：Ctrl+Alt+Backspace
但是Centos7下默认不可用，想要恢复可以参考[这里](http://mazaoliang.blog.163.com/blog/static/1384550932010101625214254/)。
3. 关于个人文件夹：
Linux是多用户，多任务的操作系统，每个人都会有自己的工作目录，这个目录是用户完全可以掌控的，称为用户个人主文件夹。
一般在目录：计算机/home/用户账号；
如我的目录在：计算机/home/kenshine下。
4. X Window 与命令行模式的切换（VMwaver下的Centos7）：
切换到命令行:Ctrl+Alt+f2/f3/f4/f5/f6/f7
切换到图形界面:Ctrl+Alt+f1
若曾经在命令行使用init跳到图形界面，就会进入图形界面
5. 在命令行模式下登陆后输入`startx`可以切换到X Window窗口
但是需要满足以下几点:
	- tty7(Ctrl+Alt+f7)没有其他窗口软件在运行
	- 必须安装了X Server能顺利启动的X Window
	- 最好有窗口管理员
	- 启动X Window所必须要的服务（X Font Sever,XFS）等必须要先启动
6. Linux启动时可以选择纯文本或是窗口环境。与运行等级相关。
Linux默认提供了7个Run level给我们使用:
		0：关机，机器关闭
		1：单用户模式
		2：多用户，无网络连接
		3：多用户,启动网络连接，标准模式
		4：用户自定义
		5：多用户,具备图形界面
		6：重启
最常用到的是run level3与run level5。在各种模式间切换需要init这个命令，例如： 
		init 0 : 关机 
		init 3 : 进入命令行模式 
也可以更改 /etc/inittab 这个文件的内容改变Linux开机时默认使用的环境,但是**CentOS-7不支持这种修改方式**。
Centos7使用以下:
		systemctl set-default multi-user.target  //设置成命令模式
		systemctl set-default graphical.target  //设置成图形模式
具体Centos7修改默认开机环境可以参考[这里](https://jingyan.baidu.com/article/ea24bc39960fa0da62b331e0.html)。
7. 登录：
使用用户名密码登录即可。输入密码时不会显示字符，连`***`都没有。
最好使用普通账户登录，root权限太大容易玩脱。一般只有需要动用到系统功能修改时才会转换身份为root。
如果想要添加新用户或忘记密码可以参考：
[Centos7关于用户的几个问题解决方案](/Centos7关于用户的几个问题解决方案.md)
注意:Centos7默认的数字键盘和是关闭的！！
登录后会看到一行像这样的文字：
![登录后文字](/img/00009Linux学习2-1.jpg)
其中kenshine是我的用户名，bogon是我的主机名，~是用户主文件夹，
root用户的~为/root，我的~指向home/kenshine。
$是普通用户提示符，root用户提示符为#。
8. 注销Linux(登出)：
	$ exit
不是关机，是退出，执行后回到未登录状态。

---
## 2.在命令行模式下执行命令
1. 命令行基本结构与概念：
结构如下：
		$ command [-option] paramater1 paramater2
			命令     选项      参数1       参数2
有下列规定：
	- 行命令中第一个输入的部分必须是命令或者是可执行文件
	- []通常不存在于与实际命令中，通常会加`-`，如-h,若使用完整参数名则用`--`，如--help。
	- 后面的两个参数是command的参数
	- 几个部分间用空格隔开，不管隔多远都当成一个空格
	- 按下Enter键命令自动执行
	- 命令太长需要换行时，在本行末尾加上`\`然后下一行继续写
	- 在Linux中区分大小写，cd与CD是不同的
2. 列出“用户主文件夹（~）下的"所有隐藏文件与相关的文件属性":
		$ ls -al ~
		$ ls -a -l ~
		$ ls 			-al ~
他们的效果是相同的。多少个空格都只算一个。
显示日期与时间：
		$ date
		
		$ Date
		$ DATE
其中只有第一个支持，其他两个找不到命令。
3. 语言的支持:
很多时候发现输入命令之后显示的结果是乱码，可能是编码问题，在命令行输入:
		$ echo $LANG
可以查看目前的编码，zh_CN为中文，en_US为英文，Centos7好像默认是英文。
可以通过以下代码修改：
		$ LANG=en_US/zh_CN
但是修改仅对这一次有效，若要一直有效，可以通过`nano`(后面会介绍)修改`/etc/sysconfig/i18n`内的LANG（Centos5.x）。
4. 基础命令操作--date/cal:
显示日期与时间：
		$ date
显示日期与时间（格式化）：
		$ date +%Y/%m/%d       <==如：2018/3/10年月日
		$ date +%H:%M:%S       <==如：18:27:14时分秒
可以看到参数前也可以加`+`号。
列出这个月日历（今日所在处会有反白表示）：
		$ cal
列出今年（2018）年整年的月历情况：
		$ cal 2018
查询今年9月的月历：
		$ cal 9 2018
基本格式为：
		$ cal [[month] year]
若月份超过12，则会提示不合法。
5. 基础命令操作--bc(计算命令)
使用`$ bc`就进入到了计算环境，此时会等待你的输入。运行`quit`可退出bc环境。以下是bc支持的基本操作:
+加；-减；*乘；/除；^指数；%余数。
默认显示结果为整数，若要显示小数可以使用`scale=number`设置，注意是在bc环境中设置。如显示三位小数：`scale=3`。
可见有些命令行执行后会自己返回，有些命令行则不会。
6. 三个极其重要的热键：
	- Tab:命令补全或文件补全(按两次tab)
			$ ca[tab][tab]
	这样写在第一个字符后会出现一些完整的命令提示：
			$ ls -al ~/.bash[tab][tab]
	这样接在一串命令第二个字符后则是文件补全，会出现许多选项
	- Ctrl+C:中断目前程序的按键
	输入了错误的命令或参数，是程序一直运行可以用Ctrl+C停止，可以用$ find/参数来测试。
	注意：若正在执行很重要的程序千万别贸然用Ctrl+C。
	- Ctrl+D：代表文字输入结束，相当于exit。

---
## 3.man page,info page
1. `$ [tab][tab]`直接连按两次tab可以查看所有命令（可能Centos5.x可以，实测Centos7不为所动）
2. man page简介
man是manual(操作说明)的简写，所以要查看某命令的详细用法就可以用man。
如要查看`date`的详细用法可以这样写:
		$ man date
进入man环境后，可以用空格翻页，按下q键来离开。以上代码运行结果顶部会出现一个这样的字符：
		DATE(1)
3. man page数字的含义：
	- 1:**一般用户在shell环境中可以操作的命令和可执行文件**
	- 2:系统内核可以调用的函数与工具等
	- 3:一些常用的函数与函数库，一般为c的函数库（libc）
	- 4:设备文件的说明，通常在/dev下的文件
	- 5:**配置文件或者是某些文件的格式**
	- 6:游戏
	- 7:惯例与协议，如Linux文件系统，网络协议等
	- 8:**系统管理员可用的命令**
	- 9:与kernel有关的文件
	- = - = - = - = - = - = - = - = - = - = - = - = - =
	以上1，5，8非常重要，可以使用man 7 man的方式获取7的详细说明，其他数字相同。使用man null时，会发现null其实是一个设备文件：null(4)
4. man page的组成：
	- NAME：数据名称的说明，剪短的命令
	- SYNOPSIS：简短的命令执行语法简介
	- DESCRIPTION：较为完整的说明，这部分得好好看
	- OPTION：针对SYNOPSIS中有列举的所有选项的说明
	- COMMANDS：环境内部可以使用的命令（如cb的scale等）
	- FILES：这个程序或数据所使用或参考或连接到的某些文件
	- SEE ALSO：此命令或数据有关的其他说明
	- EXAMPLE：一些可以参考的案例
	- BUGS：是否有相关的错误
	- = - = - = - = - = - = - = - = - = - = - = - = - =
	一般先看NAME,再看DESCRIPTION,再看OPTION,最后看SEE ALSO。
5. man page环境内的操作：
	- 空格键：向下翻一页
	- [Page Down]：向下翻一页
	- [Page Up]：向上翻一页
	- [Home]：去到第一页
	- [End]：去到最后一页
	- /String：向下查询String字符串
	- ?String：向上查询String字符串
	- n/N：n继续查询向上到向上，N反向查询向上到向下
	- q：退出man page
6. 查询某些和man page有关的说明文档：
查看出现man字符的说明文档：
		$ man -f man
查看带有特定数字的man page
		$ man 1 man
查询与man相关（不管完不完整）的说明
		$ man -k man

---
## 4. Info Page，其他有用文件及nano
1. Info Page简介
Linux额外提供的一种在线求助的方法，用法和man差不多
与man一下子输出所有信息不同，Info Page会分段，且每个页面中有类似网页的"超链接点击可以跳到不同界面，这种独立页面称为**节点**"。Info Page将说明分为多个节点，每个节点都有定位与链接。
2. Info Page环境中的按键:
	- 空格/[Page Down]：向下翻一页
	- [Page Up]：向上翻一页
	- [Tab]：在节点之间移动，有节点的地方通常会有*号显示
	- [Enter]：光标在节点上是，进入该节点
	- B：移动光标到第一个节点处
	- E：移动光标到最后一个节点处
	- N：前往下一个节点
	- P：前往上一个节点
	- U：向上移动一层（与Enter相对）
	- S(/)：在info page中查询
	- H：显示求助菜单
	- ?：命令一览表
	- q：退出，结束这info
3. 其他有用文档：
在/usr/share/doc目录下有很多在线帮助文档
4. 简单的文本编辑器nano:
Linux会有非常多文本编辑器，最重要的是vi,而nano是最简单的
直接使用`$ nano 文件名.后缀名`就可以打开旧文件或新建新文件。
进入后底部会提示很多操作，其中^代表Ctrl，如果要获取完整说明可以在nano界面按下Ctrl+G或者F1。以下是常见的操作:
	- Ctrl+G：获取帮助
	- Ctrl+X：离开nano软件，在底部会提示你是否保存
	- Ctrl+O：保存文件，有权限就可以保存
	- Ctrl+R：从其他文件读取数据，将某个文件的内容贴在文本中
	- Ctrl+W：查找字符串，很有帮助
	- Ctrl+C：说明目前光标所在处行号与列号信息
	- Ctrl+ ：直接输入行号，快速移动到该行
	- Alt+Y：校正语法功能开启或关闭
	- Alt+M：支持用鼠标来移动光标

---
## 5.正确的关机方法
1. 不正常关机的后果:
	- 可能有多人在你的电脑上工作，若就此关机，可能会导致他人无法工作
	- 可能造成文件系统的损毁，
2. 正常情况下关机要注意下面几件事:
	- 查看系统的使用状态：
	`$ who`：查看目前在线的还有谁
	`$ netstat -a`：查看网络联机状态
	`$ ps -aux`：查看后台执行的程序
	- 通知在线用户关机时刻：
	使用shutdown的特别命令功能
	- 正确关机命令的使用，后面会将
3. Linux关机一般要root账户才能执行，但是Centos允许用户在图形化界面关机，但要输入root密码。所以还是用root比较好。
4. 数据同步写入磁盘：**sync**
最好多执行几次，虽然目前shutdown/reboot等都会自动调用，但是还是多执行几次。
		# sync
		# sync; sync; sync
5. 惯用关机命令：**shutdown**
	- shutdown可以完成以下操作:
	自由选择关机模式；设置关机时间；自定义关机消息；可以选择仅发出警告而不关机；是否要用fsck检查文件系统。
	- shutdown的语法:
			# /sbin/shutdown [-t 秒] [-arkhncfF] 时间 [警告消息]
	参数:
	-t sec：t秒后关机（其余单位均为分）
	-k：不是真的关机，仅发出警告
	-r：常用：将系统的服务停掉后就重启
	-h：常用：将系统服务停掉后就关机
	-n：不经过init程序，直接使用shutdown的功能关机
	-f：关机并开机后强制略过fsck磁盘检查
	-F：系统重启后强制略过fsck磁盘检查
	-c：取消已经在进行的shutdown内容
	时间：一定要加，否则会切到单用户登录界面
	- shutdown的使用例子:
			# /sbin/shutdown -h 10 'I will shutdown after 10 mins'
			告诉大家我会在10分钟后关机
			# shutdown -h now
			立即关机，now相当于0时间
			# shutdown -h 20:35
			系统在今天20：35分关机，若执行时已经超过20：35分，则隔天执行
			# shutdown -h +10
			10分钟后自动关机
			# shutdown -r now
			系统立刻重启
			# shutdown -r 30 '666'
			系统30分钟后重启，并提升所有在线的用户
			# shutdown -k 'guanji le'
			警告所有用户，但不会关机
6. 重启、关机：**reboot,half,poweroff**
其实只要记住shutdown和reboot就行了，但是poweroff简单。
		# sync; sync; sync; reboot
更多关于reboot/half/poweroff的用法可以使用man来查询
7. 切换执行等级：
执行等级在1-6有介绍，共有7个；
其中0为关机，6为重启。
可以通过
		# init 0
		# init 6
达到关机或者重启的目的

---
## 6.开机问题排解
1. 文件系统错误：
划分分区
目前看不太懂，以后再说
2. 忘记root密码:
书上的Centos5的对Centos7无效，具体参考：
[Centos7关于用户的几个问题解决方案](/Centos7关于用户的几个问题解决方案.md)

---
## 7.学习总结：
`$ exit`：登出，退出
`$ ls -al ~`：列出“用户主文件夹（~）下的"所有隐藏文件与相关的文件属性"
`$ echo $LANG`：查看当前的编码
`$ LANG=en_US/zh_CN`：暂时修改当前编码
`$ date`：显示日期与时间
`$ date +%Y/%m/%d`：显示日期与时间（格式化）：年月日
`$ date +%H:%M:%S`：显示日期与时间（格式化）：时分秒
`$ cal`：显示本月日历
`$ cal [[month] year]`：显示year年month月的日历
`$ bc`:进入计算环境
`scale=number`：在bc环境中设置显示几位小数
`$ ca[tab][tab]`：命令补齐
`$ ls -al ~/.bash[tab][tab]`：文件补齐
`$ man date`：查看date的说明文档
`$ man -f man`：查看出现man字符的说明文档：
`$ man 1 man`：查看带有特定数字的man page
`$ man -k man`：查询与man相关（不管完不完整）的说明
`$ info date`：查看date的info page（分支型帮助文档）
`$ nano 文件名.后缀名`：打开旧文件或新建新文件。
`$ who`：查看目前在线的还有谁
`$ netstat -a`：查看网络联机状态
`$ ps -aux`：查看后台执行的程序
`# sync`：数据同步写入磁盘
`# /sbin/shutdown [-t 秒] [-arkhncfF] 时间 [警告消息]`：关机/重启
`# shutdown -h now`:立即关机
`# shutdown -r now`:立即重启
`# /sbin/shutdown -h 10 'I will shutdown after 10 mins'`：告诉大家我会在10分钟后关机
`# shutdown -k 'guanji le'`：警告所有用户，但不会关机
`# sync; sync; sync; reboot`：关机
`# init num`：切换执行等级为num,0关机，6重启

---