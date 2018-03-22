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
		#:执行的第几个命令
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
		`# declare -a variable`
数组赋值：
		`# variable[n]=值`
读取数组：
		`# echo "${var[1]},${var[2]}..."`
简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00017%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A06-09.jpg)

**3)与文件系统及程序的限制关系：ulimit**
1. 环境：
如果有多个人同时登录同一个系统，且每个人都打开了很多文件，使用了很多资源，那么系统就会炸掉。
这时就可以使用`ulimit`来限制用户的某些系统资源，包括：
打开文件的数量，可以使用CPU的时间，可以使用的内存总量等
2. 使用方法：
		# ulimit [-SHacdfltu] [配额]
		-H：严格的设置
		-S：soft limit,警告的设置


---