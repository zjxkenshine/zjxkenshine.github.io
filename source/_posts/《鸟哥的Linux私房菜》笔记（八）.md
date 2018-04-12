---
title: 《鸟哥的Linux私房菜》笔记（八）：Linux账号管理与ACL权限设置
date: 2018-04-09 10:27:59
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第14章：Linux账号管理与ACL权限设置

2. 学习重点
	- 账号管理
	- 权限规划及ACL
	- 用户身份的切换
	- 用户之间的信息传递

3. 注意：
这章都需要root权限

---
## 1.Linux的账号与用户组
**1)用户标识符：UID和GID**
1. 我们登录Linux主机时，输入的是账号，但是Linux系统是通过GID(组id)和UID（用户id）来区分的。
每个用户登录都至少会取得这两个ID。
2. 文件如何判别所有者和所在组也与GID和UID有关。
3. 可以使用root账号修改/etc/passwd文件中的普通用户的ID,这时所有的和该用户ID有关的数字也跟着改变了。
但是不能随便更改，否则会导致很多操作无法进行。

**2)用户账号：**
1. 登录时系统的处理过程：
	- 在`/etc/passwd`和`/etc/group`中比对是否有该用户的UID和GID。
	- 比对`/etc/shadow`中该UID的密码
	- 成功登录，进入shell管控
2. `/etc/passwd`文件结构介绍
cat一下发现有很多的用户，每一行代表一个，很多都是系统自带的（bin,root等），最好不要随便删除：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-01.jpg)
以第一行`root:x:0:0:root:/root:/bin/bash`为例，被冒号`:`分割为了7个字段：
	- 1账号名称：root,用来对应UID
	- 2密码：x,因为该文件所有字段都可以读取，为了安全就放到/etc/shadow文件了
	- 3UID:0
	- 4GID:0,与`/etc/group`有关。
	- 5用户信息说明列：root,解释账号意义
	- 6主文件夹:/root
	- 7获取的shell:/bin/bash
3. `/etc/shadow`文件结构介绍：
Linux为了保障用户信息安全，将用户密码放到`/etc/shadow`文件下并加了很多加密参数，cat一下如下图所示：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-02.jpg)
同样以冒号`:`作为分隔符，分为了9段，以root账号的密码字段为例：
		root:$6$WmR7M2ls44XFUhYB$Xs6Q46aC6EdOLB.a/wa2ExTCI8d5Jy1lVlkkTyI/xQ.4Jn79wNCSD8PplsBPzGWuXfhR.K4bZALAzrc2WTfrO.::0:99999:7::
各个子段的含义：
	- 1账号名称：root,和/etc/passwd对应
	- 2密码：后面很长的一大段，经过加密的密码，在该字段前加上!或者*使密码长度增加会使密码暂时失效。
	- 3最近更改日期：空，说明没修改过，一般是一个5位数的数字，代表1970年1月1日到现在的天数。
	- 4密码不可变更的天数：0，表示随时可以更改。也可以设置成大一点，防止别人更改。
	- 5密码必须重新更改的天数：99999，表示可以不修改，如果是20代表需要在20天之内修改，否则变为过期账号。
	- 6密码过期前几天提示：7表示会在99992天是给你个警告提示你账号快过期了
	- 7密码更改的宽限时间：空，表示不宽限，到那一天就过期。过期的意思就是下次登录的时候必须要设置新密码，不是失效。
	- 8密码失效日期：空，表示用不失效，如果设置为一个数字，过了那个时间后，哪个账号就永久不可以用了，区别过期。
	- 9保留字段：为新功能保留的。
4. 忘记密码之后的两种可行方法（3种）：
一般用户忘记直接找管理员解决。
root用户忘记：
	- 方法1：进入维护模式，单用户模式
	- 方式2：以Live CD挂载后修改/etc/shadow，将密码清空，然后不用密码登录后使用passwd修改
	- 方式3：重装系统

**3)有效用户组和初始用户组：groups,newgrp**
1. `/etc/group`的文件结构：
/etc/group就是用来记录GID和组名的对应的。
使用cat查看如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-03.jpg)
以第一行的`root:x:0:`为例，共有4个字段：
	- 用户组名称：root
	- 用户组密码：x,记录在/etc/gshadow中
	- GID:0
	- 此用户组支持的账号名称：空，如果想要将账号填入该用户组，只需要将用户名加到这里就可以了。一个用户可以添加到多个用户组。
2. 初始用户组与有效用户组：
用户拥有的GID在登录时都会加载，称为初始用户组。
在命令行使用`# groups`,查看所有用户组，其中的第一个就是有效用户组：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-04.jpg)
执行创建等操作时以这个用户组为准。
3. 切换有效用户组：
使用如下命令：
		# newgrp 用户组名
所切换的用户组必须是已经在的用户组。
4. `/etc/gshadow`有效用户组：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-05.jpg)
主要有以下四个字段，以第一为例：
	- 用户组名：root
	- 密码列：空，开头为！表示无合法密码，无用户组管理员
	- 用户组管理员的账号：空
	- 用户组的所属账号

---
## 2.账号管理
**1)系统管理员创建修改与删除用户**
1. useradd：
基本语法：
		# useradd [-u UID] [-g 初始用户组] [-G次要用户组] [-mM] [-c 说明栏] [-d 主文件夹绝对路径] [-s shell] 用户账号名
		-u：后面接的是UID，一组数字，直接指定一个特定的UID给这个账号
		-g：初始化用户组名
		-G：该账号可以加入的用户组
		-M：强制!不创建用户主文件夹（系统账号默认不创建）
		-m：强制!创建用户主文件夹(用户账号默认创建)
		-c：/etc/passwd第五列的说明内容
		-d：创建某个目录成为主文件夹，使用绝对路径。
		-r：创建一个系统账号，UID会有限制
		-s：要使用的shell默认为/bin/shell
		-e：接一个日期表示失效日期，格式为YYYY-MM-DD
		-f：shadow第七个字段，是否失效，0为立即过期，-1为永远不失效
系统默认给我们设置了很多选项，只要简单的使用`# useradd 用户名`就可以了，此时Centos会默认帮我们处理以下项目:
--*在/etc/passwd里面创建一行与用户相关的数据，包括UID,GID主文件夹等*
--*在/etc/shadow里面填入密码，但是没有密码*
--*在/etc/group里面加入一个和账号名称一模一样的用户组*
--*在Home下创建一个和用户名称一样的目录作为用户主文件夹*
如果指定用户组创建，且用户组已存在则不会创建和用户组同名的用户组。
用户主文件夹目录：/home/账号名称
2. useradd参考文件：
使用`useradd -D`即可查看默认的配置：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-06.jpg)
该数据是由`/etc/default/useradd`调出来的。
简单介绍每行的意思：
		GROUP=100		--默认用户组为100
		HOME=/home		--默认用户主文件夹所在位置
		INACTIVE=-1		--密码失效日（过期日）shadow中的第七列
		EXPIRE=			--账号失效日，不可使用，shadow中的第八列
		SHELL=/bin/bash		--默认的shell
		SKEL=/etc/skel		--用户主文件夹的内容数据参考目录
		CREATE_MAIL_SPOOL=yes	--帮用户创建邮件信箱
UID/GID/密码参数的默认参考地址：/etc/login.def
cat一下有很多的注释，去掉注释大概是这样的：
		[root@localhost ~]# cat /etc/login.defs
		MAIL_DIR	/var/spool/mail    --默认的邮件信箱放置位置
		
		PASS_MAX_DAYS	99999   --多久修改密码天数
		PASS_MIN_DAYS	0		--多久不可修改密码天数
		PASS_MIN_LEN	5		--密码最短长度，已被pam模块代替，无效
		PASS_WARN_AGE	7		--过期前几天警告
		//用户及用户组ID的规定
		UID_MIN     1000
		UID_MAX     60000
		SYS_UID_MIN    201
		SYS_UID_MAX   999
		GID_MIN        1000
		GID_MAX        60000
		SYS_GID_MIN    201
		SYS_GID_MAX    999
		
		CREATE_HOME	yes     --是否主动创建
		UMASK  077			--用户主文件夹创建的umask，表示是700
		USERGROUPS_ENAB yes		--删除用户时是否连带删除用户组
		ENCRYPT_METHOD SHA512 	--密码是否经过加密处理
3. passwd修改密码:
useradd创建的用户账号默认是冻结的，无法登陆因为没有设置密码，需要使用passwd来设置密码：
		# passwd [--stdin]  <--所有人都可以用来修改自己的密码
		# passwd [-l] [-u] [--stdin] [-S] [-n 天数] [-x 天数] [-w 天数] [-i 日期] 账号
		--stdin：将前一个管道的数据作为密码输入
		-l：lock,会在/etc/shadow第二列最前面加上!是使密码失效
		-u：与-l相对，unlock,解除账号锁定
		-S：列出密码相关参数，即shadow文件内的大部分信息
		-n：多久不可修改密码的天数
		-x：多久内必须要改动密码
		-w：提示天数
		-x：shadow第7个字段，密码失效的日期
可以直接使用`$ passwd`来按提示一步步修改自己的密码。
root用户可以使用`# passwd 账号`逐步修改该用户账号的密码。
还有`--stdin`参数，修改账号密码只需要如下方式：
		# echo "密码" | passwd --stdin 账号
但是可以在history中找到你的密码，所以要慎用，只在shell script需要创建很多账号时使用。
禁用与解封：`# passwd -l/-u 账号`
简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-07.jpg)
4. chage查看更详细的密码参数：
和passwd -S类似但是更加详细。语法如下：
		# chage [-ldEImMW] 账号名
		-l：列出该账号详细的密码参数
		-d：接日期，修改shadow的第3个字段（最近修改密码的时间），格式为YYYY-MM-DD
		-E：接日期，修改。。。的第8个字段(账号失效日)，格式为YYYY-MM-DD
		-I：接天数，对应第7个字段，密码失效日期
		-m：接天数，对应第4个字段，保留日期
		-M：接天数，对应第5个字段，需要多久进行修改
		-W：接天数，对应第6个字段，失效前警告日期
如列出kenshine用户的密码参数：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-08.jpg)
5. usermod修改账号数据
对账号的相关数据进行微调可以直接到/etc/passwd和/etc/shadow进行微调也可以使用usermod命令，参数如下：
		# usermod [-cdegGlsuLU] username
		参数
		-c：接账号说明，/etc/passwd第5列的说明
		-d：接账号的主文件夹，..第6列
		-e：接日期，YYYY-MM-DD,shadow第八列,失效时间
		-f：后面接天数，shadow第七列
		-g：接初始用户组，passwd第4个字段
		-G：接次要用户组，修改的是group文件的数据
		-a：与-G和用可以增加次要用户组
		-l：接账号名称，修改账号名
		-s：接shell实际文件表示使用的shell
		-u：接UID，最好不要随便修改	
		-L：冻结账户的密码
		-U：解冻密码
书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-09.jpg)
6. userdel
删除用户的相关数据，包括：
用户账号密码参数，用户组相关参数，用户个人文件夹
语法:
		# userdel [-r] 用户名
		-r：连同用户主文件夹一起删除
测试：创建一个新用户然后删除
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-10.jpg)

**2)用户功能:finger,chfn,id的使用**
上面的命令都是系统管理员才能使用的命令，一般用户也可以修改账号数据以及查询等。
1. finger查阅用户相关的信息
大部分都是/etc/passwd文件里的。基本用法：
		# finger [-s] username
		-s：仅列出用户的账号，全名，终端机代号与登录时间等
		-m：列出与后面接的账号相同者。
如果没有finger命令可以使用下面的命令自己安装(需要联网)：
		# yum -y install finger
一般用户使用`$ finger`来查看当前账号的信息。
具体的信息一般都看的懂，就不多写了。
2. chfn个人属性修改
一般用不到，简单的参数介绍：
		# chfn [-foph] [账号名]
		-f：你的全名
		-o：办公室号码
		-p：办公室电话
		-h：家里的电话
也可以直接使用`$ chfn`来修改自己的相关信息，一开始会让你输入密码确认，然后就可以修改了。
3. chsh修改自己使用的shell
使用方法：
		$ chsh [-ls]
		-l：列出目前系统上可用的shell
		-s：设置修改自己的shell
无论是chfn和chsh都有修改/etc/passwd的权限，所以该文件的权限为SUID。
4. id查看自己或者某人相关的UID/GID
有很多参数不过都不要记只要用id就可以全部列出：
		# id [username]
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-11.jpg)

**3)用户组的新增与删除：**
1. groupadd新增用户组
语法：
		# groupadd [-g gid] [-r] 用户组名
		-g：后面跟GID来直接给予GID
		-r：新建系统用户组，与/etc/login.defs中的GID_MIN有关。
最好使用`group =r 组名`的方式创建。
2. groupmod修改组参数
与usermod类似，基本语法：
		# groupmod [-g gid] [-n group_name] 用户组名
		-g：修改GID
		-n：修改组名
但是最好不要随意改动，避免出错。
3. groupdel删除用户组
		# groupdel 用户组名
如果有账号以该用户组为初始用户组则不能删除，所以删除之前得先确认。
可以删除该用户或者修改用户的GID，然后再进行删除。
4. gpasswd:用户组管理员功能
系统管理员太过繁忙，可以设置用户组管理员来协助管理该用户组。
用户组管理员可以管理那些账号可以加入或者移除该用户组。
系统管理员使用gpasswd创建用户组管理员：
		# gpasswd groupname
		# gpasswd [-A user1,user2...] [-M user3,user4...] 组名
		# gpasswd [-rR] groupname
		参数：
		没有任何参数表示给用户组设置一个密码
		-A：将该组的主控权交给后面的user
		-M：将这些用户加入该组
		-r：删除这个组的密码
		-R：使这个组的密码失效
用户组管理员的功能（系统管理员也能使用）：
		# gpasswd [-ad] user 组名
		-a：添加user到这个组
		-d：将user从这个组删除

---
## 3. 主机的具体权限规划：ACL的使用
**1)ACL的简介与启动**
1. ACL简介：
ACL全称Access Control List，提供了传统的owner,group,others的rwx权限之外的具体权限设置。
可以针对单一用户，单一文件或者目录进行rwx权限的设置。
2. ACL的启动：
ACL是传统的UnixLike支持的系统的额外支持项目，所有要使用ACL必须要文件系统的支持。可以使用mount查看，查看方法如下：
		# dumpe2fs -h 文件系统目录 |grep acl
文件系统目录可以使用`# mount`或`# fdisk -l`查看。
如果没有安装则需要重新挂载：
		# mount-o remount,acl /
这样就可以了。
基本现在的Linux都是支持的了。

**2)ACL的设置：getfacl,setfacl**
获取ACL支持后就可以设置和查看ACL了。
1. setfacl：设置acl
		# setfacl [-bkRd] [{m|x} acl参数] 目标文件|目录
		-m：设置后续的acl参数给文件使用，不能和-x合用
		-x：删除后续的acl参数，不能和-m合用
		-b：删除所有的ACL设置参数
		-k：删除ACL默认参数
		-R：递归设置ACL，子目录也会被设置
		-d：设置默认的acl参数，只对目录有效
2. getcal：查看acl
参数和setfacl相同，简要用法就是：
		# getcal 一堆参数 目标文件|目录
3. acl参数不同的情况设置方式：
	- 针对特定用户的参数：`u:[用户列表]:[rwx]`
	如：`# setfacl -m u:[kenshine]:[rwx] aaa`
	表示设置kenshine用户对aaa文件有rwx权限
	- 针对用户组的设置参数：`g:[用户组列表]:[rwx]`
	如：`# setfacl -m g:[kenshine]:[rwx] aaa`
	- 针对有效权限mask的设置方式：`m:[rwx]`
	用户或组所设置权限必须要小于mask的设置。mask在使用getcal查看用户组的acl设置时可以看到。
	如mask为r-x则设置为rwx会出错。

---
## 4.用户身份的切换
**1)为什么要切换用户身份：**
1. 平时操作系统应该使用一般账号，比较安全
2. 用较低的权限启动系统服务，
有时候需要以某些系统账号来进行程序的执行，如创建Redis用户来启动redis软件等。
3. 有些软件不能以root账号登录。

**2)su：最简单的切换命令**
1. 可以进行任何形式的身份切换。基本语法如下：
		# su [-lm] [-c 命令] [username]
		-：单纯使用减号，代表使用login-shell的变量读取方式来登录系统
		-l：与-类似，后面加要切换的用户名字，也是loginshell方式，需要登录
		-m：和-p一样的，使用目前环境的设置，而不读取新用户的配置文件
		-c：仅进行一次的命令，-c后面可以加上命令
2. 切换到root目录：`$ su`或者`$ su -`，这是临时切换用户。
但是以前一中方式未加-号，则是相当于一种环境，只是该UID有了root权限一样。读取的方式为non-login shell的方式,这种方式有很多原本的变量不会改变。
而后一种情况则是会将用户信息也一起改变，所以推荐使用-号。
不管是哪种情况，只要使用exit都可以退出su的环境。
3. 完整切换用户root：`$ su - root`或者`$ su -l root`，会将用户的USER/PATH/MAIL等一并切换过去。
4. 测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-12.jpg)
5. 如果只有一个要在另一个用户执行的命令时，可以使用-C参数，如：
		$ su - -C'执行的命令'
6. su的缺点：
root切换到其他用户不需要密码，但是其他用户之间的切换需要那个用户的密码，所以很麻烦，而且root密码会泄露。这时就需要sudo了，它只需要输入用户自身的密码，甚至可以不用密码。

**3)sudo：**
1. 并非所有人都能够执行sudo这个命令，只有/etc/sudoers内的用户才能够执行sudo这个命令。
2. sudo命令的用法：
		sudo [-b] [-u 新用户账号] [命令]
		-b：将后续的命令让系统自动执行，并不与目前的shell产生影响。
		-u：后面接要切换的用户，不加则默认为root
		-i：改变用户对命令使用权限的命令
还有很多的参数，详情可以man sudo。
这是目前版本的使用方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-13.jpg)
一般使用`# sudo -i -u 用户名`来强制切换。
3. sudo默认只有root能够使用，但是root使用sudo不需要密码。
4. 其他用户要想使用sudo命令必须要使用root权限使用visudo来设置用户。
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-14.jpg)
需要在root下加上用户名。
或者在%wheel改为用户所在的用户组。
visudo只是使用vi将sudors文件调出来而已。
5. `root ALL=(ALL) ALL`的含义
root：用户账号
ALL：登录者的来源主机名，第二个字段。
(ALL)：可切换的身份
ALL：可执行的命令，**必须是使用绝对路径编写的**
6. 通过别名设置visudo
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-15.jpg)
!表示失效，在修改时会比较方便。
7. sudo的时间间隔问题：
使用一次sudo输入密码之后，后面短时间内再次使用sudo不会再让你输入密码。
一般是5分钟之内。
8. `sudo`搭配`su`使用：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-16.jpg)
这样这几个用户就可以直接使用`$ sudo su-`来切换到root用户，而且只需要输入自己的密码就可以了。

---
## 5.用户特殊shell和PAM模块
**1)使用者的特殊shell,/sbin/nologin**
1. 系统账号的shell就是使用的/sbin/nologin。系统账号是不需要登录的所以给它这个不要登录的合法shell，而且这个shell也不能够登录。
如果使用su等方式切换到这个系统账号则会报错(如mail)
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-17.jpg)
2. 这个报错信息时可以设置的，使用vi编辑/etc/nologin.txt文件即可。
		# vi /etc/nologin.txt
这样下一次用户想要登录该系统账号时就会显示这里面编辑的信息。

**2)PAM模块简介**
1. 过去的Linux想要对一个用户进行认证需要输入账号密码，然后自行编写程序来判断密码是否正确，会导致一台Linux主机上有多个认证程序。PAM模块就是为了解决这个问题的。
2. PAM模块可以说是一套应用程序接口，它提供了一连串的验证机制，只要用户将验证阶段的需求告知PAM后，PAM就能够回报用户验证结果（成功或者失败）
3. PAM用来验证的数据称为模块，有很多个模块，每个模块的作用都不相同。
模块可以自己设置。

**3)PAM模块的设置**
1. 调用passwd命令后PAM的执行流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-18.jpg)
重点就是那个/etc/pam.d内的配置文件
2. 使用cat查看/etc/pam.d/passwd文件的内容：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-19.jpg)
可以看到分为了三个字段：**验证类型，控制标准以及PAM模块-参数**。
3. 第一个字段，验证类别，主要分为四种：
`auth`：使用密码验证用户身份
`account`：验证是否有用户的使用权限
`session`：用于记录用户登录与注销时的信息
`passwd`：密码的意思，提供验证的修订工作
这4个通常是有顺序的，但也有例外
4. 第二个字段，验证的控制标志：
主要也分为四种：
`required`：成功则带有success标志，失败则带有fail标志，但是成功或者失败都会继续后面的验证流程。有利于登录日志，最常用。
`requisite`：失败则回报原程序failure标志并且终止后面的程序继续执行，成功则带有success标志继续执行。
`sufficient`：成功则回报success终止执行，失败则带failure标志继续执行，和requisite相反。
`optional`：大多用在显示信息，非验证。
要注意的是上图的include:
调用后面的文件（/etc/pam.d/system-auth）来进行类别的验证。
5. pam的验证流程如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-20.jpg)

**4)常用的模块：**
1. 使用cat查看/ect/pam.d/login以及/ect/pam.d/system-auth
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-21.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-22.jpg)
2. 可以看到使用了很多的pam模块，详细的情报可以在以下文件获得：
/ect/pam.d/\*：每个文件的各别pam的配置文件
/lib/security/\*：pam文件的实际放置目录。
/ect/security/\*：其他pam的配置目录
/usr/share/doc/pam-\*：详细的pam说明文档
3. 常见的几个模块：
`pam_securrtty.so`：限制系统管理员只能从安全的终端机登录
`pam_nologin.so`：限制一般用户是否能够登录主机，如果存储在该文件则不能登录
`pam_selinux.so`：先将SELinux关闭，等到验证通过后再开启
`pam_console.so`：当系统出现问题时使用该模块处理一些权限问题，使用户能通过特殊的终端接口顺利登录系统。
`pam_loginuid.so`：验证uid
`pam_env.so`：设置环境变量
`pam_UNIC.so`：功能非常多，可用于验证阶段的认证，可用于账号许可证管理，可以管理日志等等
`pam_cracklib.so`：验证密码强度。
4. login的pam处理流程：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-23.jpg)

---
## 6.Linux主机上用户信息的传递
**1)查询用户：w,who,last,lastlog**
1. 查询目前已经登录在系统的用户，通过w,who来操作。
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-24.jpg)
2. 查询最近登录的用户：last,lastlog
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-25.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-26.jpg)
3. 上面所显示的pts/0等指的是终端接口。

**2)用户对谈：write,mesg,wall**
1. write：直接传递信息
		# write 用户账号 [用户所在端口]
以Ctrl+D来结束输入。
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-27.jpg)
此时的接收账号的显示：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-28.jpg)
不管该用户在干什么都会接受到这个消息。
直接按回车就可以退出消息接收。
2. 可以选择拒接一般用户的消息：
		$ mesg n
会提示无法发送消息。
开启接收一般用户的消息：
		$ mesg y
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-29.jpg)
但是来自root的消息仍然会接收到。
3. root用户群发消息，对所有用户广播：
		# wall "message"
如：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-30.jpg)

**3)用户邮箱信件：mail**
1. write,wall需要用户在线才能接收到，而使用邮箱mailbox则能在用户不在线时发送邮件。mailbox一般放在/var/spool/mail里面，一个账号有一个mailbox文件。
这里存放的都是未读的邮件。
2. 发送邮件：
		# mail 用户名 -s "邮件标题" 
		//写文件内容
		.    -- 以小数点结束
		Cc：写副本，寄给别人，Enter结束
可以使用数据流重定向来代替键盘的输入：
		# mail 用户名 -s "邮件标题" < 文件名
3. 而接收用户可以直接使用mail进入邮箱环境进行各种操作：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-31.jpg)
4. mail环境的各种操作：使用?就可以看到
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-32.jpg)
可以使用首字母来操作。
5. 常用的mail操作：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-33.jpg)

---
## 7.批量创建账号
一般不推荐手动批量新建账号。
**1)账号检查工具：**
1. pwdck,grpck
`# pwdck`：所有用户的检查，检查/etc/passwd站好配置文件内的信息与实际主文件是否相等，passwd文件和shadow文件是否匹配等。
`# grpck`：所有用户组的检查。
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-34.jpg)
2. pwconv
只是将passwd文件第二列的密码移动到shadow上而已，如果是正常使用useradd添加的用户则使用该命令`# pwconv`时不会进行任何操作。
3. pwunconv
与上述相反切会将shadow文件删除，没有备份的时候使用后果非常严重，慎用。
4. chapsswd
经常使用在批量创建账号的时候，会将未加密的密码读入，使用SHA512加密后写到shadow文件中。

**2)书上的创建范例：**
因为不太重要且比较麻烦我就不测试了，书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00033%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A08-35.jpg)

---
## 8.重要命令总结
`# newgrp 用户组名`： 切换有效用户组

```# useradd [-u UID] [-g 初始用户组] [-G次要用户组] [-mM] [-c 说明栏] [-d 主文件夹绝对路径] [-s shell] 用户账号名```
`# useradd 用户名`： 添加新用户
`# useradd -D`： 查看默认新建用户的配置
`# passwd [--stdin]`： 所有人都可以用来修改自己的密码
`# passwd [-l] [-u] [--stdin] [-S] [-n 天数] [-x 天数] [-w 天数] [-i 日期] 账号`
-->root用户用来给一般用户改密码，设置密码
`# echo "密码" | passwd --stdin 账号`： 管道方式
`# passwd -l/-u 账号`： 禁用与解封账号
`# chage [-ldEImMW] 账号名`： 查看账号的详细信息
`# usermod [-cdegGlsuLU] username`： 修改账号的信息
`# userdel [-r] 用户名`： 删除用户(-r：连同用户主文件夹一起删除)

`# finger [-s] username`： 查阅用户的相关信息
`# yum -y install finger`： 安装finger工具
`# chfn [-foph] [账号名]`： 个人信息的设置（办公室电话等）
`$ chsh [-ls]`： 修改自己的shell
`# id [username]`： 列出自己或与username相关的UID和GID

`# groupadd [-g gid] [-r] 用户组名`： 新增用户组
`# groupmod [-g gid] [-n group_name] 用户组名`： 修改用户组参数
`# groupdel 用户组名`： 删除用户组

	//root用户设置用户组管理员
	# gpasswd groupname
	# gpasswd [-A user1,user2...] [-M user3,user4...] 组名
	# gpasswd [-rR] groupname

`# gpasswd [-ad] user 组名`： 用户组管理员的功能

	//设置acl
	# setfacl [-bkRd] [{m|x} acl参数] 目标文件|目录
	-m：设置后续的acl参数给文件使用，不能和-x合用
	-x：删除后续的acl参数，不能和-m合用
	-b：删除所有的ACL设置参数
	-k：删除ACL默认参数
	-R：递归设置ACL，子目录也会被设置
	-d：设置默认的acl参数，只对目录有效

`# getfacl [-bkRd] [{m|x} acl参数] 目标文件|目录`： 查看acl

`# su [-lm] [-c 命令] [username]`： 用户身份切换，使用对方密码
`$ su - -C'执行的命令'`： 切换到root用户且只执行一次命令
`sudo [-b] [-u 新用户账号] [命令]`： 用户身份切换，使用自己的密码

`# w`,`# who`： 查看哪些用户在线
`# last`,`# lastlog`： 查看用户的上次登录时间

`# write 用户账号 [用户所在端口]`： 发送交流信息
`$ mesg y/n`： 开启，拒接消息
`# wall "message"`： 全体发送广播
`# mail 用户名 -s "邮件标题" `： 发送邮件
`# mail 用户名 -s "邮件标题" < filename`： 数据流重定向发送邮件

`# pwdck`：所有用户的检查，检查/etc/passwd站好配置文件内的信息与实际主文件是否相等，passwd文件和shadow文件是否匹配等。
`# grpck`：所有用户组的检查。

---