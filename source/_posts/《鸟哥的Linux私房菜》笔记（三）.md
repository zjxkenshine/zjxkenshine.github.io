---
title: 《鸟哥的Linux私房菜》笔记（三）：文件权限以及目录/文件的配置、管理
date: 2018-03-10 22:28:12
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第6章：文件的权限与目录的配置
第7章：Linux文件与目录管理

2. 学习重点
- 文件的权限
- 文件的属性
- Linux常见的目录
- 文件权限管理
- 文件查找

---
## 1.用户与用户组
1. Linux一般将文件可存取访问的身份分为3个类别：owner,group,others
且三种身份都各有read,write,excute（执行）等权限。
2. 用户与用户组：
不同的用户可以分到不同的组，如用户1，2分到A组，3，4分到B组，A对于12来说就是group,34对于12来说就是others。
一个用户可以归属于一个或多个用户组。
3. root用户:
有无限的权限，凌驾于所有用户之上。
4. Linux用户身份与用户组记录的文件（默认情况下）：
	- 记录系统上的账号与一般用户及root的信息：/etc/passwd文件内
	- 个人密码：/etc/shadow
	- 所有组名：/etc/group

---
## 2.文件的权限与属性（最好切换为root用户）
1. 一个非常重要的命令：`# ls`，用来显示文件名与相关属性。
如执行：` # ls -al `
就可以显示所有文件的详细权限与属性（包括文件名开头为"."的隐藏文件）
2. 执行上述命令后会出现诸如此类的列表：
		dr-xr-x---. 4 root root 4096 Mar 10 22:13 .
		-rw-r--r--. 1 root root 176  Dec 29 2013  .bash_logout
		[1]        [2] [3]  [4] [5]      [6]      [7]
		类型和权限  连接 丨 用户组 丨     修改日期   文件名
                      所有者   文件容量
	- [1]列代表了文件的类型和权限
			-rwxrw----
	可以分割为：
			 -  rwx  rw-  ---
			[A] [B]  [C]  [D]
			r:read/w:write/x:excute/-:无此权限
		- 其中[A]部分代表了文件的类型：
			- d:目录
			- -:文件
			- |:连接文件
			- b:设备文件里可供存储的接口设备
			- c:设备文件里的串型端口设备：键盘鼠标等（一次性读取设备）
		- [B][C][D]部分分别代表了：
		B:文件所有者权限(owner)
		C:同用户组权限(group)
		D:其他非本用户组的权限(others)
	- [2]列表示有多少文件名连接到此节点（i-node）：
	每个文件都会将它的权限与属性记录到文件系统的i-node中，第8章细讲。
	其中最少为1个，即为文件。也可以多于一个：目录。
	- [3]列表示这个文件（或目录）的所有者账号
	- [4]表示这个文件的所属用户组
	- [5]表示这个文件的容量大小，默认单位为B
	- [6]文件的创建日期或最近修改日期
	太过久远只会显示到日期，可以通过使用带参数ls显示完整格式：
			# ls -l --full-time
	- [7]文件名：
	以"."开头的为隐藏文件，可以使用`# ls -a`去查看
	- 更多ls的用法：`# man ls`或`# info ls`.
3. 文件权限的重要性： 
	系统保护功能，保证数据安全，实现团队合作共享
	**在修改linux文件目录属性之前，一定搞清楚什么数据是可变的，什么数据是不可变的。**
4. 改变文件属性与权限
	- 常用的改变文件的所有者，用户组及文件权限的命令：
	`# chgrp`：改变文件所属用户组
	`# chown`：改变文件所有者
	`# chmod`：改变文件的权限
	- 改变所属用户组:chgrp
	`# chgrp [-R] groupname direname/filename ...`：用户组必须是在/etc/group/下存在的才行，否则会报错误。
	[-R]参数表示递归持续更改，目录下所有文件，目录都更改到新用户组。用于目录的修改。
	如：
			# chgrp groupA install.log
	将install.log文件修改到groupA用户组
	- 改变文件的所有者:chown
	`# chown [-R] 用户名 文件或目录`：用户必须是存在/etc/passwd中的。
	- 需要改变文件的所有者时，一般先copy一份，再把复制的给他。
	`# cp 源文件 目标文件` 复制一份源文件到目标文件
	示例：
			# cp .bash .bash_copy
			# ls -al .bash*      查询以.bash开头的文件信息
			# chown -R kenshine .bash_copy
	- 改变文件权限：chmod
	用数字代表权限（9个3组），各权限的数字对照表为:
	r：4
	w：2
	x：1
	-：0
	如`rw-r--r--`的权限数字是644，`rwxr-----`权限数字为740。
	命令格式:
	`chmod [-R] 权限数字 文件名`
	- 用符号类型改变文件权限（上面是数字）:
	`chmod u/g/o/a +/-/= r/w/x 目录名/文件名`
	u/g/o/a:user,group,othre,all
	+加入/-除去/=设置   rwx读写执行
5. 修改目录与文件权限的注意事项:
	- 文件:
	w写权限可以对文件新增，编辑名字和修改文件内容，但是**不包括删除**
	文件是否可执行是由是否x权限决定的，和文件名及后缀无绝对的关系。
	- 目录:
	r:读取目录结构列表（$ ls）
	w:新建新文件与目录，删除已经存在的文件或目录（不管它权限如何），将已存在的文件或目录重命名，转移该目录下的文件目录位置。
	x:**能否进入该目录内**
	如果在某目录下没有x权限，你就无法切换到该目录下，也就无法执行该目录下的任何命令，即使拥有r权限。所以至少要有r权限和x权限才能开发浏览。
	**w权限不能随便给**
	- 几个命令(后面会有详细介绍):
	`# mkdir dirname`：新建目录
	`# touch dirname/filenam`：新建空文件
	`# su - kenshine`：从root用户切换到kenshine用户
	root到普通用户不用密码，反过来则需要
	`# cat ~/.bash`：将用户主目录下的.bash文件内容读取出来（用于读取普通文件）
	`# last dirname/filename`：用于读取特殊文件（使用cat会乱码）
6. Linux文件种类：
	- 普通文件:权限列第一个字符为[-]
		- 纯文本文件，Linux上最多的类型
		- 二进制文件，大部分可执行文件
		- 数据格式文件:data file，如登录时会将登录数据记录在/var/log/wtmp下
	- 目录：第一个属性为[d]
	- 连接：类似windows下的快捷方式
	- 设备与设备文件:
	与系统外设与存储相关的文件，通常放在/dev目录下
		-块设备文件:存储数据可以随机访问，成组设备，如硬盘等,属性为[b]
		-字符设备文件:一些串行端口设备，一次性设备，如键盘鼠标，属性[c]
	- 套接字：数据接口文件，常被用于网络上的数据连接，第一个属性[s],通常在/var/run可以看到这类文件。
	- 管道:第一个属性为[p]
	- 设备文件非常重要，不要随意修改。
7. Linux扩展名：
文件可不可执行与第一列10个属性有关，和扩展名无关。但可执行不代表执行成功。一般还是会加上扩展名表示文件类别:
	- `*.sh`：脚本或批处理文件，因为批处理文件是用shell写得
	- `*Z,*.tar,*.zip,*.tar.gz,*.tgz`：压缩文件
	- `*.html,*.php`：网页相关文件
8. 文件名长度与文件名限制:
	- 文件长度限制:
		- 单一文件最大容许文件名225个字符
		- 包含文件完整路径名称及目录(/)的完整路径名限制为4096个字符
	- 文件名限制:
	最好不要使用特殊字符，不要用.，-，+-开头

---
## 3.Linux目录配置
1. Linux目录的配置标准：FHS
详细关于FHS以及目录树的介绍参考博客:
[文件系统层次标准FHS的详细介绍](http://www.cnblogs.com/lifeinsmile/p/4280223.html)
以及[FHS目录配置下，常见的几个问题及解答](http://www.cnblogs.com/lifeinsmile/p/4280342.html)
2. 三层目录定义:
	- / (root,根目录)：与开机系统有关
	- /usr：与软件的安装执行有关
	- /var：与系统的安装运作过程有关
3. 常用目录:
	- 五个与开机有关的重要目录：
		- /etc：配置文件
		- /bin：重要执行文件
		- /dev：所需要的设备文件
		- /lib：执行文件所需的文件与内核所需的模块
		- /sbin：重要的系统执行文件
	- 根目录下几个很重要的目录:
		- /boot：存放开机会用到的文件
		- /home：用户主目录（~dmstsai代表dmstsai的用户主目录，~代表自己）
		- /lib：开机会用到的函数库
		- /media：可删除设备
		- /mnt：与media相同，现只用来暂时挂载
		- /root：管理员主文件目录
		- /opt：第三方软件放置目录，最好是distribution提供的
		- /ser：网络服务数据目录
		- /proc：虚拟文件系统
		- /sys：也虚拟文件系统，记录内核相关信息
	- /usr相关几个重要目录：
		- /usr/bin/：绝大部分用户可以使用的命令，与开机无关的
		- /usr/lib/：各应用程序的函数库，目标文件及不被一般用户惯用的执行文件或脚本。
		- /usr/local/：自己下载安装的软件放这，非distribution提供的。
		- /usr/sbin/：非系统正常运行所需要的系统命令，常见的是网络服务命令
		- /usr/src/：源码放这里，内核源码放在/usr/src/Linux/下
	- /var相关的几个重要目录：
		- /var/cache/：应用程序运行中产生的一些暂存文件
		- /var/lib/：程序执行过程需要使用的数据文件放置目录，如MySQL放到/var/lib/mysql/下
		- /var/lock/：同步资源放置处
		- /var/log/：登录文件
		- /var/run/：放置启动后的服务/程序的PID
4. 查看Centos：
`# uname -r`：查看实际的内核版本

---
## 4.目录与路径
1. 绝对路径与相对路径
	- 绝对路径：由根目录（/)写起
	- 相对路径：由当前目录出发，相对于当前目录。如../表示上级目录下
		- .：代表此层目录
		- ..：上一层目录
		- -：前一个工作目录(从下层目录cd上来的则去到下层)
		- ~：目前用户所在的主文件夹
		- ~kenshine：用户kenshine的主文件夹
2. 目录的相关操作:
		# cd     切换目录
		# pwd    显示当前目录
		# mkdir  新建一个新的目录
		# rmdir 删除一个空目录
	- `# cd`简单介绍：
		`$ cd 绝对路径/相对路径`
	提示：可以利用Tab键快速补全目录文件。
	- `# pwd`显示出当前工作的目录：
	`# pwd -p`：显示当前工作路径的真实目录，非连接目录
	- `# mkdir dirname`创建新目录：
	`# mkdir -mp`:-m配置权限数字，-p创建所需要的目录如：
	`# mkdir -p test1/test2/test3/test4`：若test1-3不存在则会自动创建
	`# mkdir -m 755 test1`创建并配置test1目录权限为drwxr-xr-x。
	新创建的目录中会有两个隐藏文件.和..代表本目录与上级。
	- `# rmdir dirname`删除空目录
	`# rmdir -p dirname`删除空目录并连同上级空目录一起删除
	如`# rmdir -p test01/test02`,若cd到test01下再执行，则test01不会被删除
4. 关于执行文件路径的变量：$PATH
	- FHS标准告诉我们一些命令如`# ls`的真实路径是/bin/ls,但是在任何地方只要用ls就能执行了，这是因为环境变量PATH的帮助。
	- 如ls,系统会查询每个PATH目录下名为ls的可执行文件，先找到的先执行
	- 可以使用`# echo $PATH`命令查询PATH中定义了哪些变量。
	echo显示，打印出。$表示后面接的是变量
	注意：使用普通用户查询比使用root查询缺少包含sbin的目录
	- 如何将一个目录加到可执行文件的查询路径PATH中(如/root/test01)：
	`# PATH="$PATH":/root/test01`
	- 若本身在/root/bin内未配置PATH如何调用ls:
	`# ./ls`或`# /root/bin/ls`不能直接使用`# ls`
	- 本目录"."最好不要放到PATH中

---
## 5.文件与目录管理
1. 查看文件与目录：ls
		# ls [-aAdfFhilnrRSt] 目录名称
		# ls [--color={never,auto,always}] 目录名称
		# ls [--full-time] 目录名称
常用参数介绍：
		-a:列出全部文件，连同隐藏文件（常用）
		-A:列出全部文件，连同隐藏文件但是不包括"."与".."这俩目录
		-d:仅列出目录本身，而不是列出目录内的数据（常用）
		-f:直接列出结果而不进行排序
		-F:根据文件目录信息给予附加的数据结构，如：
		   *代表可执行文件；/代表目录；|代表FIFO文件；=代表socket文件
		-h:将文件容量以人类较易读的方式列出来。（KB,GB等)
		-i:列出icode码
		-l:列出长数据串，包含文件属性与文件等级等数据（常用）
		-n:列出UID和GID而非用户和用户组
		-r:将排序结果反向输出
		-R:连同子目录内容一起列出来
		-S:以文件容量大小排序而不是用文件名排序
		-t:依时间排序而不是用文件名
		--color=never:不要给文件依据特性给予颜色显示
		--color=always:显示颜色
		--color=auto:系统自动判断是否显示颜色
		--full-time:以完整时间模式显现（年月日时分秒）
		--time={atime,ctime}输出访问时间或改变权限属性时间（ctime）而不是内容更改时间（atime）
ls最常用的参数还是`-l`,可以用`$ ll`效果等同于`$ ls -l`，至于为什么会这样涉及到了bash shell的alias（别名）功能。
如执行`# ls /etc`会显示很多蓝色的目录和白色的一般文件。
注意：如果你在文件夹test01中，使用`# ls test01`将会没有效果，要使用`# ls`或者`# ls .`
2. 复制、删除与移动:cp,rm,mv3
3. 复制：cp(copy)
`cp`除了单纯的复制功能，还可以创建连接文件（即快捷方式），对比新旧并予以更新，以及复制整个目录等。用法如下:
		# cp [-adfilprsu] 源文件（source） 目标文件（distination）
		# cp [options] source1 source2 source3 ... directory
		注意:多个源文件最后必须是目录
参数说明:
		-a:相当于-pdr的作用，-p-d-r稍后介绍（常用）
		-p:连同文件属性一同复制过去而不是使用默认属性（备份常用）
		-d:若源文件为连接文件的属性，则复制连接文件的属性而并非文件本身
		-r:递归持续复制常用于目录的复制行为（常用）
		-f:强制复制，若文件已存在而无法使用，则删除后再尝试复制
		-i:若目标已存在，则会询问是否覆盖（常用）
		-l:进行硬连接的连接文件创建而非文件本身复制
		-s:复制成复制成符号链接文件，即快捷方式。
		-u:若destination比source旧才更新destination。
在默认条件下，cp的源文件与目标文件的权限是不同的，目标文件所有者通常是命令操作者。在-a下可以复制所有（包括权限，属性等）。
几个例子:
		# cp -r /etc/ /tmp  复制/etc/目录下的所有内容到/tmp，如果不加-r则会失败
		# cp -a /root/test01 .  将test01文件复制到自身（完整复制）
		# cp -u /root/test01 test01  两个文件有差异时才会复制（备份常用）
		
		# cp -s test01 test02  软连接复制
		# cp -l test01 test03  硬连接复制
		若使用ls -al查看会发现：
		硬连接test03的文件名仍然为test03
		软连接test02的文件名是像这样的：test02 -> test01
不同身份者执行这个命令会有不同的结果产生，尤其是-a与-p参数：
		$ cp -a /var/log/wtmp /tmp/vbird_wtmp
		可以复制 除用户文件组 以外的所有属性，有时该身份无权限。
复制之前需要了解：
	>是否需要完整保留源文件的信息?  -a
	>软连接或硬连接  -s  -l 
	>源文件是否为链接文件  -d(不加则会复制实际文件)
	>源文件是否为特殊文件  -a
	>源文件是否为目录  -r
4. 删除，移除文件或目录：rm(remove)
		# rm [-fir] 文件或目录
参数说明：
		-f:强制删除，忽略不存在的文件，不会发出警告
		-i:互动模式，再删除前询问用户是否操作
		-r:递归删除，常用在目录删除，非常危险
示例：
		# rm -i test*  删掉本目录下以test开头的所有文件/目录，*是通配符，代表从0到无穷多个任意字符。
		# rm -r dir01  递归循环删除dir01目录下的所有子项但是会一直询问
		# \rm -r dir01 忽略掉alias指定参数（-i）,不询问，但是很危险
		# rm ./-aaa-  文件名以-开头的直接rm -aaa-会被当成参数，所以需要这样删除或者# rm -- -aaa-
为了文件误杀，很distributions都已经默认加入了-i这个参数
5. 移动文件与目录，或改名：mv(move)
		# mv [-fiu] source destination（目标文件）
		# mv [options] source1 source2 ....direction
		多个文件最后一个目标一定是目录
参数说明:
		-f:强制移动，若目标已存在，则不会询问直接删除
		-i:目标存在，询问
		-u:update,目标存在且source比较新，更新
示例：
		重命名：# mv A B   将A重命名为B，需要B不存在才行
		移动:# mv file dir
6. 关于重命名：
Linux中有一个名为rename的命令，可以用来更改大量的文件文件名，具体用法man rename
7. 获得文件名与目录名：
`# basename 完整文件名`获得文件名
`# direname 完整文件名`获得目录名
如：
		# basename /etc/sysconfig/network    //network
		# dirname /etc/sysconfig/network     ///etc/sysconfig/

---
## 6.文件查看与创建
1. 简单介绍各个文件查看命令：
	- cat:由第一行开始显示文件内容
	- tac:从最后一行开始显示
	- nl:显示的时候顺便输出行号
	- more:一页一页地显示文件内容
	- less:与more相似，但是更好用，可以向前翻页
	- head:只看头几行
	- tail:只看后几行
	- od:以二进制方式读取文件内容
2. 直接查看文件内容的几个命令:cat,tac,nl
	- cat
			# cat [-AbEnTv] filename
			-A:相当于-vET，可以列出一些字符
			-v:列出一些看不出来的字符
			-E:将结尾的换行符（$）显示出来
			-T:将Tab键以^T的方式显示
			-b:列出行号，仅针对非空白行列出行号
			-n:列出行号，空白行也有行号
	有一些特殊字符要显示的话就要用-a等参数
	cat是Concatenate(连续的简写)，主要功能是将内容连续显示在屏幕上。
	但是当行数超过40行时会来不及查看。
	- tac
			# tac filename
	与cat刚好反过来，由最后一行向第一行显示
	- nl:显示的时候顺便输出行号
			# nl [-bnw] filename
			-b:指定行号指定方式，主要有两种:
				-b a:空行也显示行号（类似cat -n）
				-b t:空行不显示行号(类似cat -b)
			-n:列出行号表示的方法，有三种:
				-n ln:行号在屏幕左边显示
				-n rn:行号在右边显示，不加0
				-n rz:行号在右边显示，加0
			-w:行号字段占用位数
	如：`# nl -b a -n rz /etc/issue
	注意：若只使用-b，则会报错。如`# nl -b issue`会报错
3. 可翻页查看的查阅:more与less
	- more
			# more filename
	当文件行数大于显示行数时，光标会在最后一行停留并显示目前百分比，可以按下面的键进行操作:
		- 空格键（Space）：下一页
		- Enter：下一行
		- /字符串:在这个显示当中向下查询"字符串"关键字
		- : f：立刻显示出文件名以及现在显示的行数
		- q：立刻离开more，不再显示该文件内容
		- b或Ctrl-b：往回翻页，不过只对文件有用，对管道无用
		- n：重复查询
	- less
			# less filename
	比more的用法更有弹性，可以向上翻动与向上查询
		- 空格键：向下一页
		- [PageDown]：向下翻动一页
		- [PageUp]：向上翻动一页
		- /字符串：向下查询
		- ?字符串：向上查询
		- n：重复前一个查询
		- N：反向重复查询前一个查询（向上变向下）
		- q：离开less
4. 数据选取:head,tail
	- head
			# head [-n number] filename
			-n number:代表显示几行，默认为10行
	显示前20行:
			# head -n 20 issue
	若总共100行，则显示前90行可以这样写:
			# head -n -10 issue
	-10代表最后10行不显示。
	- tail
			# tail [-n number][-f] filename
			-n number:显示最后几行，默认最后10行
			-f:持续监测，新加入的数据都会显示出来，直到按下ctrl+c离开
	显示最后20行:
			# tail -n 20 issue
	从第100行开始显示:
			# tail -n +100 issue
	持续监测issue的更新：
			# tail -f issue
5. 读取为进制文件:od（查阅非纯文本文件）
		# od [-t 参数] filename
		-t后面可以加各种参数达到不同输出效果:
			a：利用默认的字符来输出
			c：使用ASCII字符来输出
			d[size]：使用十进制来输出数据，每个整数占用size bite
			f[size]：利用浮点数来输出数据，每个数占用size bite
			o[size]：用八进制来输出数据，每个数占用size bite
			x[size]：用十六进制来输出数据，每个整数占用size bite
6. 修改文件时间或创建新文件:touch
	- 每个文件在Linux下都会有很多记录时间的参数:
		- modification time(mtime):
		文件内容修改时会更新这个时间，属性或权限更改时不会更新
		- status time(ctime):
		文件状态改变时会更新这个时间，属性或权限更改也会更新
		- access time(atime):
		文件内容被取用时会更新这个时间。如使用cat读取时就会更新这个时间
	- 使用ls时获取的时间时mtime,也就是内容上次被更改的时间
	- touch
			# touch [-acdmt] filename
			不加参数:创建一个空白文件
			-a：仅修改atime读取时间为当前时间
			-c：仅修改ctime状态更新时间为当前时间
			-d：后面接时间，不用当前时间，也可以用--date="日期或时间"
			-m：仅修改mname内容修改时间为当前时间
			-t：后面接时间，格式为[YYMMDDhhmm]
	- `# touch filename`
	若文件不存在，则创建一个新文件，且三个时间默认更新为当前时间	

---
## 7.文件与目录的默认权限与隐藏权限
1. 文件默认权限：unmask
	- 表示用户在新建文件或目录时的权限默认值(目前所在目录的权限)
			# unmask
			# unmask -S
	上面一种方式会输出4个数字如：
			0022
	第一个数字是特殊权限，后面三组就是ugo的权限，022代表的权限数字是755，是表示没有哪项权限的。所以权限是这样的：rwxr-xr-x.
	下面加了-S参数的，表示以字符显示，结果：
			u=rwx,g=rx,o=rx
	- 使用unmask修改默认权限
			# unmask 反权限数字
	如一个原本权限为rwxr--r--的权限改为rwxrwxr-x可以这样写：
			# unmask 002
2. 文件隐藏属性设置与修改：chattr
chattr属性只能在[Ext2/Ext3文件系统](http://blog.csdn.net/u012317833/article/details/24537265)上面生效。
		# chartt [+-=][ASacdistu] dirname/filename
		+:增加某一个参数，其他原本存在的参数不动
		-:删除某一个参数，其他原本存在的参数不动
		=:仅有后面接的参数，其他参数失效
		(下面是常用的俩参数)
		a：设置了a权限后，这个文件的数据只能增加，不能删除，不能修改只有root才能操作这个属性
		s：可以让一个文件无法（增删改查），设置连接也不行，只有root能设置这个属性
		（其他参数）
		A:设置了A后访问此文件或目录时atime不会被修改，防止IO较慢的及其过渡磁盘访问，对速度慢的计算机有帮助。
		S:一般文件是异步写入磁盘的，加上S参数会同步写入磁盘
		c:自动将文件压缩，读取再解压，存储时先压缩再存储，对大文件有很大帮助
		d:但dump程序被执行时，设置该属性可以使该文件或目录不会被dump备份。
		s:设置这个属性后，若该文件被删除，则会被从磁盘也删除
		u:与s相反，文件删除会保存在磁盘里
如：
设置与取消issue的i属性可以这样:
		# chartt +i issue
		# chartt -i issue
3. 隐藏属性的查看：lsatter
显示文件隐藏属性：
		# lsatter [-adR] 文件或目录
		-a:将隐藏文件的属性也列出来
		-d:如果接的是目录，仅列出目录本身的属性（不会列出目录内的文件名）
		-R:连同子目录属性也一起列出来
4. 文件的特殊权限：SUID,SGID,SBIT(最好看完17章程序再回来看)
	- SUID:Set UID(Set UerId标记用户id)
	权限像这样情况下就是SUID:(s将user的x代替)
			-rwsr-xr-x
	SUID的限制和特殊功能：
			1.仅对二进制程序有效（不能用于shell脚本）
			2.执行这需要对该文件有x执行权限
			3.本权限仅在程序执行过程中有效
			4.执行者将拥有此程序所有者的权限
	- SGID:Set GID（设置用户组id）
	权限像这样：(s将group的x代替)
			-rwxr-sr-x
	SGID可以标记文件和目录有效
	对于文件:
			1.SGID对二进制程序有用
			2.程序执行者对于该程序来说，需具备x的权限
			3.执行者在执行过程中将会获得该程序所属用户组的权限
	对于目录:
			1.用户对于此目录有r与x权限时，该用户能进入此目录
			2.用户在此目录下的有效用户组将会变成该目录的用户组?_?
			3.若用户在此目录下右w权限（可新建文件），则用户所创建的新文件的用户组与此用户的用户组相同。
	**对于项目开发非常有用**。
	- SBIT：(Sticky Bit)只针对目录有效
	如/temp的本身权限是：
			drwxrwxrwt
	使用SBIT有以下限制效果:
			1.当用户对于此目录有w,x，即具有写入权限时才有效
			2.当用户在该目录下创建文件或目录时，仅有自己与root能删除
	- SUID/SGID/SBIT权限设置:
			SUID:4
			SGID:2
			SBIT:1
	设置方法和普通权限相同，使用chmod或unmask如:
	`# chmod 7755`则为rwsr-sr-t`
5. 查看文件类型:
		# file filename

---
## 8.命令与文件查询
1. 命令简介
	- which:查找脚本，命令等可执行文件
	- whereis:查找特定类型文件
	- locate:简单查找文件，与whereis都是用数据库找，速度很快
	- find:多种方式查找文件，不常用，locate与locate找不到才有find找
2. 脚本,命令文件名查询:which（寻找"执行文件"）
		# which -a command
		-a:将所有PATH目录中可以找到的命令都列出，而不只是第一个
	可以查询到用别名命名过的命令。如`# which -a ls`
3. whereis:查询特定的文件
		# where [-bmsu] dirname/filename
		-b:只找二进制格式的文件
		-m:只找在说明文件manual路径下的文件
		-s:只找source源文件
		-u:查找不在上述选项中的其他特殊文件
有些文件目录未加入PATH中无法被which找到，可以使用whereis查找到。
但是有时会找到已经被删除的文件或找不到刚创建的文件，因为它是在数据库中查找的，速度很快，数据库说明见locate。
4. 模糊查找：locate
		# locate [-ir] keyword
		-i:忽略大小写差异
		-r:后面可接正则表达式的显示方式
直接在locate后面输入部分文件名就可以查找。但是使用需要注意更新数据库。
注意:locate的数据库在/var/lib/mlocate/下，但是每天只更新一次，所以刚创建的文件使用locate是查不到的。可以使用：
		# updatedb
命令更新locate数据库后再查找。
5. 多种方式查找：find
在硬盘中查找，速度不是很快：
		find [PATH] [option] [action]
			  路径    参数     额外动作
它有好几种参数:
1.与时间有关的参数：
		有-ctime,-atime和-mtime,下面以-mtime说明:
		-mtime n:n为数字，意义为在n天之前的“一天内”被修改的文件（n-1,n）
		-mtime +n:列出n天之前（不含n天）修改的文件(好久以前，n)
		-mtime -n:列出n天内修改的文件[n,0)
		-newer file:file为一个存在的文件，列出比file还要新的文件
		
		# find /home 0
		0代表现在，表示查找从现在开始24小时内所有改动的文件
2.与用户用户组有关的参数:
		- UID n:n为用户的id即uid
		- GID n:n为用户组id即gid
		- user name:按用户账号查找
		- group name:按用户组名查找
		- nouser:文件的所有者不存在/etc/passwd/中的文件
		- nogroup:文件的所在用户组不存在/etc/group中的文件
3.与文件名称及权限有关的参数:
		-name filename:查找文件名为filename的文件
		-size [+-]SIZE:查找比SIZE大+或小-的文件，c代表byte,k代表kB,
			查找大于50KB的文件:find -size +50k
		-perm mode:权限刚好等于mode的文件如-perm 7777
		-perm -mode:查找文件必须全部包括mode权限，或者说是权限大于mode
		-perm +mode:查找包含mode某一项（特殊ugo）权限的文件
		如：
		# find / -perm -mode +7000:查找包含--s--s--t权限的文件
4.其他相关参数：
		-exec command:command为其他命令，-exec后面可再接其他命令来处理查找结果
		-print:默认参数，将结果打印
如：
		# find / -perm -mode +7000 -exec ls -l {} \;
从`-perm`到`\;`为额外命令，用`{}`指代`-perm`前的查询结果，并对结果进行`ls -l`操作。

---
## 9.权限与命令间的关系（很重要）
1. 让用户进入某目录成为**可工作目录**的基本权限是什么:
	- 可使用命令：如cd切换到工作目录
	- 权限要求：至少要有x权限
	- 额外需求：若要使用ls,则需要r权限
2. 用户在某一个目录内读取一个文件的基本权限：
	- 可用命令：cat,tac,less,more等
	- 目录权限要求：对这个目录至少要有x权限
	- 文件权限需求：对这个文件有r权限
3. 用户可以修改某一文件的基本权限：
	- 可使用命令：nano,vi编辑器等
	- 目录所需权限：至少要有x权限
	- 文件所在权限：至少要有r,x权限
4. 一个用户可以创建一个文件：
目录所需权限：至少要有w,x权限
5. 让用户进入某目录并执行该目录下的某个命令的基本权限：
	- 目录所需权限：至少有x
	- 文件所需权限：至少有x

---
## 10.重要命令总结
`# ls -al `:显示文件名与相关属性(含隐藏文件)
`# chgrp [-R] groupname direname/filename ...`：修改文件/目录所在用户组，目录用-R表示递归，子目录也修改
`# chown [-R] 用户名 文件或目录`：用户必须是存在/etc/passwd中的，改变文件的所有者
`chmod [-R] 权限数字 文件名`：用权限数字改变文件权限
`chmod u/g/o/a +/-/= r/w/x 目录名/文件名`：用符号改变权限数字

`# su - kenshine`：从root用户切换到kenshine用户，root到普通用户不用密码，反过来则需要
`# uname -r`：查看实际的内核版本

`$ cd 绝对路径/相对路径`：切换到目录
`# pwd`显示出当前工作的目录：
`# pwd -p`：显示当前工作路径的真实目录，非连接目录
`# mkdir dirname`创建新目录：
`# mkdir -mp`:-m配置权限数字，-p创建所需要的目录如：
`# mkdir -p test1/test2/test3/test4`：若test1-3不存在则会自动创建
`# mkdir -m 755 test1`：创建并配置test1目录权限为drwxr-xr-x。
`# rmdir dirname`：删除空目录
`# rmdir -p dirname`：删除空目录并连同上级空目录一起删除
`# echo $PATH`：查询PATH中定义了哪些变量
`# PATH="$PATH":/root/test01`：将一个目录加到可执行文件的查询路径PATH中(如/root/test01)

`# ls [-aAdfFhilnrRSt] 目录名称`：列出文件目录属性
`# ls [--color={never,auto,always}] 目录名称`：列出的文件目录属性是否显示颜色
`# ls [--full-time] 目录名称`:列出文件目录的完整时间
`# cp [-adfilprsu] 源文件（source） 目标文件（distination）`:复制文件或目录
`# cp [options] source1 source2 source3 ... directory`:复制文件到目录
`# cp -r /etc/ /tmp `: 复制/etc/目录下的所有内容到/tmp，如果不加-r则会失败
`# cp -a /root/test01`:  将test01文件复制到自身（完整复制）
`# cp -u /root/test01 test01 `:两个文件有差异时才会复制（备份常用）
`# rm [-fir] 文件或目录`:删除文件或目录（-f:强制删除，-i:互动模式，-r:递归删除）
`# rm -i test*`: 删掉本目录下以test开头的所有文件/目录，*是通配符。
`# rm -r dir01`:递归循环删除dir01目录下的所有子项但是会一直询问
`# \rm -r dir01`: 忽略掉alias指定参数（-i）,不询问，但是很危险
`# rm ./-aaa- `：文件名以-开头的直接`rm -aaa-`会被当成参数，所以需要这样删除或者`# rm -- -aaa-`
`# mv [-fiu] source destination（目标文件）` ：移动文件到目标地点（f:强制移动，-i:目标存在，询问，-u:更新）
`# mv [options] source1 source2 ....direction`： 移动多个文件到目录
`# mv A B`: 将A重命名为B，需要B不存在才行
`# mv file dir`:移动文件到目录
`# basename 完整文件名`：获得文件名
`# direname 完整文件名`：获得目录名

`# cat [-AbEnTv] filename`:由第一行开始显示文件内容
`# tac filename`:与cat刚好反过来，由最后一行向第一行显示
`# nl [-bnw] filename`:显示的时候顺便输出行号
`# more filename`:带分页的显示
`# less filename`:more的升级版
`# head [-n number] filename`:显示头number行，默认10
`# head -n -10 issue`：最后10行不显示。
`# tail [-n number][-f] filename`:显示最后几行，默认最后10行

`# tail -n +100 issue`：从第100行开始显示。
`# tail -f issue`：持续监测issue的更新（-f:持续监测直到按下ctrl+c离开）
`# od [-t 参数] filename`:查阅非纯文本文件

`modification time(mtime)`:文件内容修改时会更新这个时间，属性或权限更改时不会更新
`status time(ctime)`:文件状态改变时会更新这个时间，属性或权限更改也会更新
` access time(atime)`:文件内容被取用时会更新这个时间。如使用cat读取时就会更新这个时间
`# touch [-acdmt] filename`:修改文件时间或穿件空白文件
`# touch filename`:创建一个空白文件

`# unmask`：目前用户所在目录的权限，反权限数字表示
`# unmask -S`：目前用户所在目录的权限，字符表示
`# unmask 反权限数字`:使用unmask修改默认权限
`# chartt [+-=][ASacdistu] dirname/filename`:文件隐藏属性的设置和修改
`# chartt +i issue`
`# chartt -i issue`:设置与取消issue的i属性
`# lsatter [-adR] 文件或目录`：显示文件隐藏属性
`# file filename`:查看文件类型

`# which -a command`:脚本命令文件查找（-a:将所有PATH目录中可以找到的命令都列出）
`# where [-bmsu] dirname/filename`:查找特定文件（b:二进制的，-m:在说明文件路径下的文件-s:source源文件-u:查找不在上述选项中的其他文件）
`# locate [-ir] keyword`：简单查找（-i:忽略大小写差异-r:后面可接正则表达式的显示方式）
`# updatedb`:更新查找数据库
`# find [PATH] [option] [action]`:在磁盘中查找，多样查找
`# find / -perm -mode +7000 -exec ls -l {} \;`:查找包含权限--s--s--t的文件并以list打印

---
