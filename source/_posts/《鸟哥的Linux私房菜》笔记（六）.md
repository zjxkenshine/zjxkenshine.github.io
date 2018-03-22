---
title: 《鸟哥的Linux私房菜》笔记（六）：认识与学习bash
date: 2018-03-22 10:11:01
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第11章：认识与学习bash

2. 学习重点

---
## 1.认识Shell,Bash
**1)硬件，内核与shell的关系：**
1. 简单来说：
用户可以通过shell来与内核通信，使内核能够控制硬件准确无误地工作。
2. 示意图：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-01.jpg)
3. 大致定义：
只要能够操作应用程序的接口都能称为shell，狭义的shell指的是命令行方面的软件，包括Bash等，广义的shell则包括图形界面的软件，因为图形界面其实能够操作各种应用程序来调用内核工作。

**2)为什么学shell:**
1.所有Linux发行版的shell都是差不多的，学好了shell就可以随意切换了。
2.远程管理时命令行界面的传输速度比较快，而且较不容易出现断线等问题。

**3)shell简介:**
1. Bash:
Bourne Again SHell:Linux使用的Shell。
shell有很多种类：
BSD,C shell,sh等。
检查一下/etc/shells会发现有很多可用的shell,必须将可用的shell写到这个文件里：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-02.jpg)
当打开命令行开始工作时，会得到一个shell，这个shell记录在`/etc/passwd`文件内。
默认为`bin\bash`。
2. bash shell 的功能：
bash shell是Linux发行版的标准shell，它有如下优点：
	- 命令记忆功能：
	本次登录执行过的命令会暂存在内存中，直到退出时会被写入~/.bash_history文件内，可以查询所做过的操作，默认可以记忆1000个。
	- 命令与文件补全功能：[Tab键]
	tab接在一串命令的第一个字后则为命令补全，否则为文件补全
	- 命令别名设置功能，可以直接设置别名：
	使用alias，如`alias lm='ls -al'`
	- 作业控制，前后台控住作业
	- 程序脚本：shell script(13章)
	- 通配符*
3. 查看bash shell内置命令：type
输入man page可以看见一大堆！！！说明，据说在里面可以找到cd，umask等命令，而这些命令就是bash的内置命令。判断一个命令是否是内置命令可以使用type命令：
		$ type [-tpa] name
		-p:后面接的name为外部命令时，显示完整的文件名
		-a:将PATH变量中能找到的name都列出来。
		-t:根据不同类型的命令显示不同的说明：
		    file:外部命令
		    alias:表示是别名设置过的命令
		    builtin:bash内置的命令功能。
		不加参数：列出命令主要使用情况
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-03.jpg)
4. 命令的换行：
在行尾输入\[Enter]来转义，到下一行开头会自动出现>号，就可以继续写未写完的命令。

---
## 2.shell的变量功能（环境变量，语系变量等）
**1)什么是变量?**
1. 变量：
用一个简单的文字代替另一个比较复杂或者容易变动的数据。
2. 变量可变性，方便：
如：Mail变量管理邮箱访问，不同的用户输入Mail命令时会访问不同的邮件目录。
3. 变量可变，但是很方便。
4. 环境变量：
系统需要一些变量来提供它的数据访问–环境变量，如PATH,HOME,MAIL,SHELL等，为了区分普通变量，一般以大写字母表示。
5. 变量可以方便程序的设计
6. 变量的定义：
变量就是以一组文字或符号等，代替一些设置或者一些保留符号。

**2)变量的显示与设置：echo和unset**
1. 变量的显示：echo
可以利用echo这个命令来显示变量，但是变量被显示时，前面必须加上字符”$”才行,也可以加上花括号{}，如显示PATH的值：
		# echo $PATH
或者：
		# echo ${PATH}
2. 变量的设置与取消：=，unset
设置变量：直接# 变量名=值就行
变量设置取消：`# unset 变量名`
下面是一个简单的测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-04.jpg)
变量未设置时默认内容为空。
3. 变量设置规则：
变量设置是需要符合以下规则，否则会创建失败：
	- 变量与变量内容以一个=来连接
	- 等号两边不能接空格符：以下都是错的
			# myname = kenshine   # myname=ken shine
	- 变量名称只能是数字或字母，但是开头不能是数字
	- 变量内容若有空格符可以用""或单引号括起来，但是
	双引号保持原有特性，而单引号则只会保留为字符串：
			# var="is $LANG" -->echo $var--> is en_US
			# var='is $LANG' -->echo $var--> is $LANG
	- 可用转义符号将特殊符号变为一般字符：#,$,!等
	- 一串命令中若要得到其他命令的信息，可以使用反引号``，如：
			# version=`uname -r`-->ceho $version-->会输出版本信息
	- 若变量为了增加内容时，可以使用$\*或${\*}来累加内容：
			# PATH="$PATH":/home/bin
	- 变量要在其他子线程进程，需要用export编程环境变量：
			# export PATH
	- 大写字母为环境变量，小写字母为用户自己设置的变量。
4. 子进程：
在当前的shell打开另一个新的shell,则哪个shell就是子进程。

**3)环境变量的功能：**
1. 使用`env`与`export`查看目前shell环境下的环境变量：
环境变量的基本功能如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-05.jpg)
使用export也是一样的效果。
2. 使用`# set`查看当前shell的所有变量
包括环境变量，接口有关的变量，用户自己设置的变量和其他变量。
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-06.jpg)
3.  命令提示符PS1：
就是常看见的`#`与`$`及前面部分的内容，但是还有很多参数，具体可以`man bash`,简单几个：
		\t:显示时间，24小时制，HH:MM:SS
		\#:执行的第几个命令
		\u:目前账号名称
		\H:完整的主机名
		\w:完整的工作目录
…还有很多
修改命令提示符：
		# PS1='[\参数...]\$'
4. 本机shell的PID:`$`
5. `?`:上个命令的回传值（0执行成功，非0不成功）
6. `export`:将自定义变量转换为环境变量
不转换切到子进程则变量不见，直到离开子进程回到父进程才能重新看到变量。

**4)语系变量（local）:**
1. 可以使用:
		# local [-a]
来查询语系变量。
2. 使用`# local`会发现有很多种变量，都是关于数字，时间，币值等各种显示，只要修改LANG或LC_ALL时其他变量就会随之改变。
		# LANG=en_US
3. 整体系统默认的语系：
定义在`/etc/sysconfig/i18n`中

---
## 3.变量的范围，修改，键盘输入及限制变量
**1)变量的有效范围：**
环境变量==全局变量
自定义变量==局部变量

**2)变量的键盘读取、数组与声明：read,array,declare**
1. 读取键盘输入的值：**read**
		# read [-pt] 变量名（variable）
		-p:后面可以接提示符
		-t:后面接等待的秒数
简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-07.jpg)
2. 变量声明：**declare或typeset**
`declare`与`typeset`功能一样，后面不加任何参数就会将变量全部列出，和set差不多：
		# declare [+-aixr] 变量名（variable）
		-a:将后面的变量声明为数组（array）类型
		-i:将后面的变量声明为整数（integer）类型
		-x:同export一样，将变量变为环境变量
		-r:将变量设置为只读变量，无法更改
		+参数:取消该参数的设置，变回默认的状态
默认类型为字符串，而且没有字符型的运算，如1/3结果为0。
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-08.jpg)
使用typeset也是一样的效果。
3. 数组（阵列）变量类型：**array**
在程序设计中，数组非常重要。
声明数组：
		# declare -a variable
数组赋值：
		# variable[n]=值
读取数组：
		# echo "${var[1]},${var[2]}..."
简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-09.jpg)

**3)与文件系统及程序的限制关系：ulimit**
1. 环境：
如果有多个人同时登录同一个系统，且每个人都打开了很多文件，使用了很多资源，那么系统就会炸掉。
这时就可以使用`ulimit`来限制用户的某些系统资源，包括：
打开文件的数量，可以使用CPU的时间，可以使用的内存总量等
2. 使用方法：
		# ulimit [-SHacdfltu] [配额]
		-H：hard limit,严格的设置，一定不能超过这个值
		-S：soft limit,警告的设置，超过则会有警告
			一般来说警告设置会比严格设置小
		-a：后面不接参数，可以列出所有的限制配额
		-c：限制内核文件的最大容量。
			某些进程发生错误时，系统会将进程中的内存写到文件中（内核文件）
		-f：此shell可创建的最大文件容量（一般为2GB），单位为KB
		-d：此进程可使用的最大断裂内存容量
		-l：可用于锁定的内存量
		-t：可使用的最大CPU时间
		-u：单一用户可使用的最大进程数量
如我的配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-10.jpg)

**4)变量内容的部分删除、替代与替换：**
1. 变量内容的部分删除：
从前向后删除：
		${变量#关键字}    从头开始符合关键字的最短数据删除
		${变量##关键字}    从头开始符合关键字的最长数据删除
从后向前删除：
		${变量%关键字}    从尾开始符合关键字的最短数据删除
		${变量%%关键字}    从尾开始符合关键字的最长数据删除
关键字中一般使用`*`通配符来代表一大段字符串。
如书本上的范例：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-11.jpg)
从后向前删也是一样的。
2. 变量的替换：
		${变量/旧字符串/新字符串}    第一个旧字符串被新字符串替换
		${变量//旧字符串/新字符串}    全部旧字符串都被新字符串替换
3. 变量设置方式：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-12.jpg)
会根据不同的情况赋予不同的值:
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-13.jpg)

---
## 4.命令别名与历史命令
**1)命令别名设置：alias,unalias**
1. 关于clean：
早期的DOS使用cls来清除屏幕上的信息，Linux则使用`# clear`来清除界面。
如果想要用`# cls`来执行清除任务，就可以用别名来设置。
2. alias设置别名
查看当前设置了哪些别名：`# alias`
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-14.jpg)
别名的设置(详情查看上图↑)：
		# alias 别名="命令 选项"
设置cls为`clear`的别名：
		# alias cls="clear"
3. 取消别名：
		# unalias 别名
如取消上述设置的cls:
		# unalias cls

**2)历史命令：history**
1. 基本用法：
		# history [n]
		# history [-c]
		# history [-raw] historyfiles
		不加参数：列出所有历史
		n:数字，列出最近的几行命令
		-c:消除当前shell的所有history
		-a:将新增的history中的命令加入historyfiles中，如果没有加historyfiles参数，则会默认添加到~/.bash_history中
		-r:将historyfiles中的记录读取出来
		-w:将目前的history记录加入historyfiles中
使用：
		$ echo $HISTSIZE
可以查看一共可以容纳多少条记录（容量）。
2. 三个执行历史命令的方法：
		# !number
		number代表命名标号，执行第几条命令
		# !command
		从后往前搜索第一个以command开头的命令并执行
		# !!
		执行上一个命令
3. 不同窗口多次登录同一账号产生的问题：
如在tt2-tt6都登陆了root账号且都执行了一些命令，那么前面退出的root记录都会被最后退出的root覆盖，只会将最后一个退出的history信息写入文件.
4. 历史命令无法记录时间

---
## 5.Bash Shell的操作环境
**1)命令的执行顺序：**
1. 命令的运行顺序：
a.以绝对/相对路径执行命令，如`# /bin/ls`
b.由alias找到该命令来执行
c.有Bash内置的（builtin）命令来执行
d.通过$PATH这个变量来找到第一个命令来执行。
2. 一个例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-15.jpg)
按照`alias-->builtin-->$PATH`这个顺序执行。

**2)bash的登录与欢迎信息：**
1. 系统的登录信息位置：
		/etc/issue
里面的变量是通过反斜杠`\`来定义的，有如下变量：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-16.jpg)
2. 提供给telnet远程登录的欢迎信息：
		/etc/issue.net
3. root用户在其他用户登录时通知信息：
		/etc/motd
不需要反斜杠，使用vim打开，把想通知用户的消息都记录下来就可以了。

**3)bash环境配置文件：**
1. 命令别名，自定义变量等在注销退出bash后就会失效，若想保留你的设置，就需要将这些设置写入配置文件才行。
2. 认识配置文件之前必须知道的两个东西：**login shell和non-login shell**
	- login shell:需要登录才能取得bash的shell称为login shell（[Ctrl]+[Alt]+F23456打开的黑窗口）
	- non-login shell:取得shell不需要不需要登录的。（图形界面登录后通过终端跳转，或者打开一个子进程`# bash`）
	- login shell：此种方式登录时，shell会重新读取/etc/profile和~/.bash_profile来应用新的环境变量。
	- non-login shell：此时shell不会读取/etc/profile和~/.bash_profile，而是读取~/.bashrc来应用新的环境变量。
3. /etc/profile(login shell才会读:第一个读取)
这是系统的整体配置，最好不要修改，了解即可。
所含变量：
		PATH:依据UID决定是否含UID目录
		MAIL:依据账号设置好的mailbox的账号名
		USER:根据用户的账号设置此变量的内容
		HOSTNAME:根据主机的HOSTNAME决定的此变量内容
		HISTSIZE:历史命令记录容量
		umask:默认root为022，普通用户为002
读取该配置文件时以下的配置文件会依次被读取：
<blockquote>
a.`/etc/profile.d/*.sh`:
定义了bash的颜色，语系，默认别名等
b.`/etc/locale.conf`:
定义了默认的语系
c.`usr/share/bash-completion/completions`:
tab的文件/命令补齐功能
</blockquote>
`login shell`环境下读取的整体配置文件只有`/etc/profile`
4. ~/.bash_profile(login shell才会读:第二个读取)
login shell在读取完整体配置文件(/etc/profile)后会依次读取下列文件：
		~/.bash_profile
		~/.bash_login
		~/.profile
只会读取一个，后面的不读取，这么多文件是为了照顾其他shell用户的习惯。
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-17.jpg)
如果~/.bashrc如果存在就会把这个文件的配置也读取进来。
5. ~/.bashrc（`non-login shell`才会主动读取）:
root用户与一般用户的会不一样。
主要有下面的功能：
&nbsp;&nbsp;&nbsp;--用户的个人设置（别名等）
&nbsp;&nbsp;&nbsp;--依据不同的UID规定umask值
&nbsp;&nbsp;&nbsp;--依据不同UID规定PS1命令提示符
&nbsp;&nbsp;&nbsp;--调用/etc/profile.d/*.sh设置
如果不小心删除，可以复制/etc/skel/.bashrc文件过来并`# source ~/.bashrc`
这是我刚配置的.bash文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-18.jpg)
6. `source`命令：
立即读入刚配置的环境变量（login shell不用注销再登录）。
立即应用刚配置的环境：`source`或小数点
		# source 配置文件
如应用我刚刚配置好的.bashrc文件可以这么写：
		$ source ~/.bashrc
或者
		$ . ~/.bashrc
7. 其他相关配置文件：
这些文件也可能会影响你的bash操作：
	- /etc/man.config：
	规定了使用man page时man page的路径去哪里寻找
	- ~/.bash_history:
	记录历史命令的地方
	- ~/.bash_logout:
	注销bash后系统做的工作

**4)设置终端机环境：stty,set**
1. stty:settinf tty,设置终端机，其实就是设置热键（快捷键），一般不建议修改。
可以使用`# stty [-a]`查看[所有]参数列表。
2. 终端环境中几个功能字符串的意义：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-19.jpg)
如想要设置[Ctrl]+h来删除字符串可以这么写：
		# stty erase ^h
这里的`^`都是代表[Ctrl]。
3. `set`可以设置整个命令的输入输出环境：
		# set [-uvCHhmBx]
		-u:未设置变量报错功能，默认未启用
		-v:信息输出时显示原始内容，默认未启用
		-x:命令执行前默认启用
		-H,h:默认启用，与历史命令有关
		-m:默认启用，与工作管理有关
		-B:默认启用，与[]的作用有关
		-C:默认不启用，使用数据流重定向等时原有的文件存在则不覆盖
最好还是保持默认就好。
常用的组合按键表：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-20.jpg)

**5)通配符遇特殊符号：**
1. 常用通配符：
		* : 0到无穷多个任意字符
		? : 一定有一个任意字符
		[]: 一定有一个在[]中的任意字符，如[a,b,c]表示一定有abc中的一个
		[-]: 括号内有-号代表在编码顺序内的所有字符。
		[^]: 表示反向选择，[^abc]有任意一个非abc的字符
2. 常见的特殊符号：
书上定义的列表：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-21.jpg)

---
## 6.数据流重定向：
**1)什么是数据流重定向：**
1. 简单的说：
数据流重定向就是讲某个命令执行后应该要出现在屏幕上的数据传输到其他地方（文件或者是设备等地方，用于保存数据）
2. 命令执行过程的数据传输情况：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-22.jpg)
3. 标准输出与标准错误输出：
standard outout标准输出：指的是命令执行后所传回来的正确信息。
standard error output标准错误输出：命令执行失败后输出的异常信息。
4. 数据流重定向可以将这些输出都重定向到其他文件或设备去，所用字符如下：
		标准输入(stdin):代码为0，使用<或<<
		标准输出(stdout):代码为1，使用>或>>
		标准错误输出（stderr）:代码为2，使用2>或者2>>
简单测试:
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-23.jpg)
若重定向的文件不存在，则会自动创建，若存在，使用`>`覆盖，使用`>>`不覆盖。
使用`2>`,`2>>`则是输出错误的信息。
5. 黑洞设备垃圾桶（/dev/null）
查找`/home`下是否有名为`.bashrc`文件，如果有，则显示，错误信息丢弃：
		$ find /home -name .bashrc 2>dev>null
6. 正确与错误信息都输入同一个文件(list):
		$ find /home -name .bashrc >list 2>&1
或者：
		$ find /home -name .bashrc &>list
7. 利用cat创建一个文件:
		# cat >~/catfile
然后可以输入，输入完后按[Ctrl]+d来离开，创建完毕。
8. 标准输入（`<`与`<<`）
相当于从键盘输入数据到文件中，只不过是从文件中读入。
**注意<与<<的功能并不相同**：
`<`表示从一个文件读入数据：
		# cat > catfile <~/log
`<<`后面跟字符串，输入这个字符串则离开输入：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-24.jpg)

**2）何时使用重定向：**
- 输出信息很重要需要保存
- 后台执行程序的输出会干扰当前执行程序
- 一些系统例行命令的执行结果保存
- 一些已知的不重要的错误信息可以丢弃
- 错误信息需要与正确信息分别输出时


**3)命令执行的判断依据：；，&&，||**
1. 想要一次性执行多个命令有两种方法：
	- 编写shell script脚本去执行（13章）
	- 使用`;`，`&&`或`||`
2. `cmd1;cmd2`：
不考虑文件相关性连续执行命令（Linux默认从左往右）
如关机前使用多个`syn`将数据同步到磁盘：
		# syn;syn;syn
3. 命令回传码$?:
若一个命令执行成功则回传`$?=0`，执行失败则传回非0的值。
4. `cmd1&&cmd2`：
若cmd1执行完毕且正确执行（$?=0）则执行cmd2,错误不执行。
5. `cmd1||cmd2`：
若cmd1执行完毕且错误执行（$?≠0）则执行cmd2,正确不执行。

---
## 7.常用命令（管道，选取，排序，双向重定向）
**1)管道命令与连续执行命令的区别：**
1. `cmd1|cmd2`:
cmd1执行完毕且正确执行时，将其标准输出作为标准输入作为cmd2的数据，然后执行cmd2.(仅能处理standard output)
如用ls-al查看文件但是太多了想用less输出可以这样写:
		# ls -al /etc | less
2. 管道命令仅会处理standard output的信息，对于错误输出无法处理；
管道命令必须能够接收来自前一个数据命令的数据成为标准输入继续处理。

**2)选取命令：cut,grep**
1. 选取信息一般来说是针对"行"来分析而不是整篇信息。
2. cut:
基本用法：
		# cut -d'分隔字符' -f fileds      <==用于分隔字符
		# cut -c 字符范围                 <==用于排列整齐的信息
		-d:接分隔符，与-f一起使用
		-f:依据-d分隔符分为数段，-f表示取第几段
		-c:以字符为单位取出字符区间
例子：
		$ echo $PATH | cut -d':' -f 5		"以：给PATH变量分段并取第五段（第3与第5：-f 3,5）
		# export | cut -c 12-		"输出所有的环境变量信息，并且每行都删除前11个字符，只保留第十二个字符及之后的
		# export | cut -c 12-20			"每行只保留第12-20的字符
3. last:
`# last`可以显示出用户登录列表，后面有用到。
4. grep:
cut是在一行中找到我们想要的信息，grep是分析一行信息，其中有我们需要的信息时，**将该行拿出来。**
基本用法：
		# grep [-acinv] [--color-auto] '查找字符串' filename
		-a：将binary文件以text文件的方式查找数据
		-c：计算找到'查找字符串'的次数
		-i：忽略大小写
		-n：输出行号
		-v：反向选择
		--color=auto：找到的关键字加上颜色显示
简单例子：
		# last | grep 'root' | cut -d ' ' -f1
		"显示登录信息的所有root
		# grep --color=auto 'MANPATH' /etc/main.config
		"找出 /etc/main.config文件中带MANPATH关键字的行并显示颜色
grep配合正则表达式还可以有更多的功能。

**3)排序命令：sort,wc,uniq**
1. sort:
可以用于排序，而且可以根据不同的数据类型进行排序：
排序与语系有关，建议先使用LANG=C让语系同一后排序。
基本语法：
		# sort [-fbMnrtuk] [file or stdin]
		-f:忽略大小写差异
		-b:忽略最前面的空格符
		-M:以月份名字来排序
		-n:使用纯数字进行排序
		-r:反向排序
		-u:uniq,相同的数据不重复排序
		-t:分隔符，默认为[Tab],作用同cut的-d
		-k:以区间（filed）进行排序
简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-25.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-26.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-27.jpg)
书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-28.jpg)
2. uniq:去重
基本用法：
		# uniq [-ic]
		-i:忽略大小写
		-c:进行计数
例子：
记录登录的总次数：
		# last | grep 'root' | cut -d ' ' -f1 | uniq -c
3. wc:记录有多少个字，多少行
基本用法：
		# wc [-lwm]
		-l：仅列出多少行
		-w:仅列出多少字（单词）
		-m:列出多少字符
如得知当前账号文件中有多少账号：
		# cat /etc/passwd | wc -l

**3)双向数据流重定向：tee**
1. 一个输出，即想将它保存处理，又想将其显示出来：
使用>,>>无法做到，除非读入那个文件，这时就可以使用tee.
2. tee的工作流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-29.jpg)
输出到屏幕的是stdout，可以让下一个命令继续处理
3. 基本语法：
		# tee [-a] file
		-a:以累加的方式重定向到file
4. 将last输出备份一份到last.list中并输出登录用户：
		# last | tee last.list | cut -d ' '  -f1

---
## 8.常用命令（字符转换，切割，参数代换和减号）
**1)字符串转换命令:tr,col,join，paste,expand**
1. tr：
可以删除掉一段信息当中的文字，或者进行文字信息的转换
用法：
		# tr [-ds] SET1...
		-d:删除信息当中的SET1这个字符串
		-s:替换重复的字段(正则表达式)
例子：
		"将/etc/passwd输出信息中的":"删除：
		# cat /etc/passwd | tr -d ':'
		"将last输出信息的所有小写字母变为大写字母：
		# last | tr '[a-z]' '[A-Z]'      "不加引号也可以
2. col:
col经常被用于将man page转存为纯文本文件方便查阅：
		# col [-Axb]
		-A:将tab键显示为^（默认）
		-x:将tab键转换为对等的空格键
		-b:文字内有反斜杠时仅保留反斜杠后面的那个字符
例子：
将col的man page转换为文本文件：
		# man col | col -b > col.man
3. **join:**
处理两个文件之间的数据。主要是将两个文件当中有相同数据的那一行加在一起。
基本用法：
		# join [-ti12] file1 file2
		-r:默认以空格符分隔数据，默认比较第一个字段的数据，若相同则连在一起
		-i:忽略大小写的比较
		-1：第一个文件用哪个字段比较
		-2：第二个文件用哪个字段来比较
书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-30.jpg)
4. paste：
直接将文件的两行粘贴在一起，中间以[tab]隔开
基本用法：
		# paste [-d] [-] file1 file2
		-d:后面可以写分隔符，默认为tab
		-:重要的参数，代表标准输入的结果（即管道前的标准输出）
例子：
		# cat /etc/group|paste /etc/passwd /etc/shadow - |head -n 3
		"三重连接并且取出连接结果的前三行
上述例子的`-`参数就代表了`cat /etc/group`的标准输出，即对于后面的标准输入。
5. expand:
就是将tab键转换成空格键：
基本语法：
		# expand [-t] file
		-t:后面可以接数字，表示将一个tab用多少个空格代替，一般是8个空格。
6. unexpand:
和expand刚好相反，将空格转换为[tab]

**2）切割命令：split**
1. 使用环境：
如果文件太大导致便携式设备无法复制，就可以切割成小文件。
2. 基本用法：
		# split [-bl] file SPEFIX
		-b:后面接想切割成的大小，可以带单位（b,k,m等）
		-l:以行数来进行切割
		SPEFIX:前导符，作为切割文件的前导文字，会以前导文字+aa,ab,ac...为文件名创建小文件
3. 例子：
将一个文件以300k一份切割并设置前导文字为aaa
		# cd /tmp;split -b 300k /etc/termcap aaa
将使用`ls -al /`输出的信息以10行为一份分为多个小文件
		# ls -al / | split -l 10 -  aaa

**3)参数代换：xargs**
1. finger命令:可以显示内容，后面例子里有
2. xargs:
产生某个命令的参数的意思。xargs可以读入stdin的数据，并且以空格符或者断行字符进行分辨，将stdin的数据分割成参数。因为以空格作为分隔符，有可能误判。
3. 具体用法：
		xargs [-0epn] command
		-0:stdin中有特殊字符，将它还原为一般字符
		-e:eof,end of file,后面可以接一个字符串，xargs碰到该字符串时停止工作。
		-p:执行每个命令的参数时都会询问用户的意思
		-n:接次数，表示一次Commad使用几个参数
		不加参数的xargs默认使用echo输出
4. 例子:
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-31.jpg)
5. **有些命令不支持管道时就可以使用参数代换，将前面的显示结果作为参数传给改名令（如finger）**。

**4)关于减号的用途：**
使用前一个命令的stdout作为这个命令的stdin.

---
## 9.重要命令总结
`$ type [-tpa] name` :判断一个命令是否是内置命令
`# echo $变量`或`# echo ${变量}` ：显示变量的值
`# 变量="$变量":累加内容` :变量累加
`# export 变量` ：将自定义变量转换为环境变量
`# env`与`# export` :查看目前shell环境下的环境变量
`# set` :查看当前shell的所有变量
`# PS1='[\参数...]\$'` :修改命令提示符
`# local [-a]` :查询语系变量

`# read [-pt] 变量名（variable）` :读取键盘输入的值存入变量中
`# declare [+-aixr] 变量名（variable）` :声明变量类型
`# declare` :列出全部变量
声明数组：
`# declare -a variable`
数组赋值：
`# variable[n]=值`
读取数组：
`# echo "${var[1]},${var[2]}..."`
`# ulimit [-SHacdfltu] [配额]` :限制用户的系统资源

	${变量#关键字}    从头开始符合关键字的最短数据删除
	${变量##关键字}    从头开始符合关键字的最长数据删除
	${变量%关键字}    从尾开始符合关键字的最短数据删除
	${变量%%关键字}    从尾开始符合关键字的最长数据删除
	${变量/旧字符串/新字符串}    第一个旧字符串被新字符串替换
	${变量//旧字符串/新字符串}    全部旧字符串都被新字符串替换

`# alias 别名="命令 选项"` ：设置别名
`# unalias 别名` :取消别名设置

	# history [n]
	# history [-c]
	# history [-raw] historyfiles
	查看历史记录

`$ echo $HISTSIZE` ：可以查看一共可以容纳多少条记录（容量）。
`# !number` :number代表命名标号，执行第几条命令
`# !command` :从后往前搜索第一个以command开头的命令并执行
`# !!` :执行上一个命令

`# source 配置文件`或`# . 配置文件` :立即应用刚配置的环境
`# stty [-a]` :查看[所有]热键参数列表
`# set [-uvCHhmBx]` ：设置整个命令的输入输出环境

	常用通配符
	* : 0到无穷多个任意字符
	? : 一定有一个任意字符
	[]: 一定有一个在[]中的任意字符，如[a,b,c]表示一定有abc中的一个
	[-]: 括号内有-号代表在编码顺序内的所有字符。
	[^]: 表示反向选择，[^abc]有任意一个非abc的字符

	数据流重定向
	标准输入(stdin):代码为0，使用<或<<
	标准输出(stdout):代码为1，使用>或>>
	标准错误输出（stderr）:代码为2，使用2>或者2>>


正确与错误信息都输入同一个文件(list):
`$ find /home -name .bashrc >list 2>&1`
或者：
`$ find /home -name .bashrc &>list`

`cmd1;cmd2` ：不考虑文件相关性连续执行命令
`cmd1&&cmd2` ：若cmd1执行完毕且正确执行（$?=0）则执行cmd2,错误不执行。
`cmd1||cmd2` ：若cmd1执行完毕且错误执行（$?≠0）则执行cmd2,正确不执行。
`cmd1|cmd2` :cmd1执行完毕且正确执行时，将其标准输出作为标准输入作为cmd2的数据，然后执行cmd2.(仅能处理standard output)

`# cut -d'分隔字符' -f fileds` :分隔后选取
`# cut -c 字符范围 ` :排列好后按字符选取
`# grep [-acinv] [--color-auto] '查找字符串' filename` :按行选取

`# sort [-fbMnrtuk] [file or stdin]` :排序
`# uniq [-ic]` :去重
`# wc [-lwm] ` :记录有多少个字，多少行

`# tee [-a] file ` :双向数据流重定向，-a:以累加的方式重定向到file
`# tr [-ds] SET1...` :删除信息中的文字
`# man col | col -b > col.man` :将col的man page转换为文本文件

`# join [-ti12] file1 file2` :将两个文件有相同数据的文件连在一起
`# paste [-d] [-] file1 file2` :直接将两个文件连在一起

`# expand [-t] file` :将tab键转换为空格
`# unexpand [-t] file` :将空格准转换为tab

`# split [-bl] file SPEFIX` :将大文件以某种标准切割成小文件

`xargs [-0epn] command` :参数代换

---