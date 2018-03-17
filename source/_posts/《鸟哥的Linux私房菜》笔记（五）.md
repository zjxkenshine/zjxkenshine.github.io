---
title: 《鸟哥的Linux私房菜》笔记（五）:文件压缩备份与vim编辑器简介
date: 2018/3/15 17:26:00 
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第9章：文件与文件系统的压缩打包
第10章：vim程序编辑器

2. 学习重点
- 压缩及打包
- 各种备份操作
- vim(很重要)

---
## 1.压缩简介已及常见的压缩命令
1. 压缩简介:
文件压缩：利用一些复杂的计算使文件占用空间变小。
解压缩：使用这些被压缩的文件数据时，将他们还原成未压缩前的样子。
压缩比：压缩前与压缩后的磁盘空间大小比。
2. 关于压缩文件的扩展名:
压缩的技术不同，解压也需要不同的技术，为了方便人们在解压时知道是哪种技术压缩的，所以适当的扩展名还是需要的,下面是常用的扩展名:
		*.Z    compress程序压缩的文件
		*.gz    gzip程序压缩的文件
		*.bz2   bzip2程序压缩的文件
		*.tar   tar程序打包的数据，并没有压缩过
		*.tar.gz  tar打包的数据，其中经过gzip的压缩
		*.tar.bz2 tar程序的打包文件，其中经过的是bzip2的压缩
3. Compress:
太老旧了，只有在非常旧的Unix系统中才会有，CentOS没有安装，需要自己手动装。但是gzip已经可以打开.Z文件了，所以使用compress没有什么意义。（我把语法抄一抄）
		# compress[-rcv] 文件或目录     //压缩
		-r:递归压缩目录下的文件
		-c:将数据输出成为标准输出，并输出到屏幕上
		-v:显示压缩后的文件信息以及压缩时的过程名变化
		# uncompress 文件.Z           //解压
4. **gzip,gcat**:
gzip是目前应用最广的压缩命令，gzip主要是想要来代替compress的，所有gzip可以用来解压Compress,zip,gzip等文件压缩的文件，gzip的简单用法如下:
		# gzip [-cdtv#] 文件名
		-c:将压缩数据输出到屏幕上，可以通过数据流重定向处理
		-d:解压
		-t:可以用来检验一个压缩文件的一致性，看看文件有无错误
		-v:可以显示出源文件/压缩文件的压缩比等信息
		-#:压缩等级-1最快但是压缩比最差，-9最慢但是压缩比最好,默认6
使用gzip压缩的文件在Windows系统中被WinRAR解压。使用例子：
		# gzip -v man.config   //压缩，会将源文件删除
		# gzip -d man.config.gz   //解压缩，会将.gz删除（不要用gunzip）
		# gzip -9 -c man.config > main.config.gz
		//将man.config以最优压缩比压缩，并保留原来的文件
使用zcat命令可以纯文本被压缩后的文件,不用解压(类似cat):
		# zcat 文件.gz
		# zcat man.config.gz   //例子
好像不能用来压缩目录与软链接文件。
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-01.jpg)
5. **bzip2,bzcat**:
bzip2就比较吊了，是为了取代gzip提供更好的压缩比而生的。压缩比比gzip还厉害，用法几乎与gzip相同，只是后缀为`.bz2`：
		# bzip2 [-cdkzv#] 文件名
		-c:降压缩过程中产生的数据输出到屏幕上
		-d:解压
		-k:保留源文件，不会删除原始文件
		-z:压缩
		-v:可以显示出源文件/压缩文件的压缩比等信息
		-#:压缩等级，-9最好最慢，-1最快最差，默认6
若解压时已经有相同的文件存在，则会解压失败，不会问你覆不覆盖。
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-02.jpg)
使用bzcat和使用gcat效果相同，都是不解压读取文件数据:
		# bzcat 文件名.bz2
还可以使用bunzip来代替bzip2 -d解压。详细用法`man bunzip`

---
## 2.打包命令：.tar
上一节的文件压缩只能进行文件的压缩，虽然可以加上-r参数，但也只是对目录内文件进行单独压缩。
1. 打包命令tar：
将多个文件或目录包成一个大文件的命令。--tar,tar可以将多个文件或目录打包成一个大文件同时还可以通过gzip/bzip2的支持，将该文件同时进行压缩。
tar的参数非常多，只介绍几个，详情man tar。
2. 文件的压缩与打包:
		# tar [-j][-z] [cv] [-f 新建的文件名] 被压缩的文件/目录
		-j:通过bzip2支持进行压缩/解压缩，此时文件名最好为*.tar.bz2。
		-z:通过gzip的支持进行压缩/解压，此时文件名最好为*.tar.gz。
		-c 目录 ：在解压时若要指定特定的目录解压就可以使用这个参数
		-v:压缩/解压过程中将正在处理的文件名显示出来
		-f filename:后接被处理（打包压缩）后的文件名，建议-f单独写
		-p:保留备份数据的原本权限与属性，常用于备份（-c）重要文件
		-P:保留绝对路径，允许备份数据之中含有根目录
简单记忆:
		# tar -jcv -f filename.tar.bz2 要被压缩的文件或者目录名称
		# tar -zpcv -f filename.tar.bg 要被压缩的文件或者目录名称
加上-p参数备份保存属性与权限是一个好习惯，简单测试:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-03.jpg)
3. tar压缩文件的查阅
		# tar [-j][-z] [-tv] [-f 新建文件名]
		-t:查看打包文件的内容，含有那些文件名，重点是查看文件名
		其他参数与压缩打包相同。
简单记忆:
		# tar -jtv -f filename.tar.bz2
		# tar -ztv -f filename.tar.gz
简单测试:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-04.jpg)
注意：上述文件压缩与打包时是默认将根目录“/”去掉的，防止出现覆盖，更加安全，若执意要加上根目录可以使用-P参数。
4. 解压：
		# tar [-j][-z] [xv] [-f 新建文件夹] [-C 目录]
		-x:解打包或解压缩的功能，可以搭配-C在特定目录打开，特别留意的是，-c,-t,-x不能同时出现在一串命令行中。
		-C 目录：用在解压时，指定特定的目录解压可以使用这个参数。
简单记忆:
		# tar -jxv -f filename.tar.bz2 -C 欲解压目录
		# tar -zxv -f filename.tar.gz -C 欲解压目录
简单测试（-C后所带的目录必须要存在才行）:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-05.jpg)
5. 解压某一特定文件的方法(未测试):
书上的例子:
		//先查找需要的文件，假设为shadow,配合|与grep选取关键字
		# tar -jtv -f /root/etc.tar.bz2 |grep'shadow'
		//再将文件解开(假设找到的是etc/shadow)
		# tar -jxv -f /root/etc.tar.bz2 etc/shadow
解开单一文件的语法:
		# tar -jxv -f filename.tar.bz2 待解压文件名
6. 打包某目录但不包括该目录下某些文件:
使用`--exclude=file`参数，可以将指定的file文件排除在压缩文件之外。
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-06.jpg)
基本语法:
		# tar -jcv -f 新建文件名 --exclude=不想加的文件1 [--exclude=...] 待打包压缩的文件或目录
7. 仅备份比某个时刻还要新的文件：
有两个参数可以使用：`--newer-mtime`和`--newer`，前者只包含了mtime,后者包含了mtime和ctime。
书上例子:
		//find找出比/etc/passwd新的文件
		# find /etc-newer /etc/passwd
		// 打包比某一时间新的文件
		# tar -jcv -f /root/etc.newer.tar.bz2 --newer-mtime=\
		  >"2008/09/29" /etc*
8. tar备份到磁带机（不知道centos7可不可以）：
tar不仅可以将数据打包成文件，还可以将文件打包到其它设备中。
想要将/home,/root,/etc备份到磁带机/dev/st0时，可以这么写:
		# tar -cv -f /dev/st0 /root /home /etc
9. 特殊应用：利用管道流命令与数据流（11章）
 将整个目录一边打包一边在/tmp中解开
		# cd /tmp
		# tar -cvf - /etc |tar-xvf -

---
## 3.完整备份工具:**dump**
1. dump简介及原理
dump除了可以备份整个文件系统(仅用于EXT2/EXT3)之外，还可以定制等级，但是需要文件系统备份。
第一次dump（默认等级为0）备份后，第二次备份时可以指定备份等级，如等级1则会和等级0比较，而备份2只会记录与第一次的差别。
2. dump备份分两种情况
 	- 当备份数据为单一文件系统:
	单一文件系统可以使用dump的完整dump功能，包括利用0-9的数个等级备份，同时备份时可以用挂载点或设备文件名进行备份。
 	- 当备份的数据只是目录，并非文件系统时:
	有限制：
	1.所有备份数据都必须在该目录下面
	2.仅能支持使用level 0,仅支持完整的备份而已
	3.不支持-u参数，无法创建/etc/dumpdates记录备份时间
3. dump语法:
		# dump [-Suvj] [-level] [-f 备份文件] 待备份数据
		# dump -W
		-S:仅列出后面的待备份数据要多少空间才能备份完毕
		-u:将这次dump时间记录到/etc/dumpdateS文件中
		-v:将dump过程显示出来
		-j:加入bzip2的支持对数据进行压缩，默认等级为2
		-level：备份等级0-9
		-f:类似tar,后面接产生的文件，可接设备文件名
		-W:列出在/etc/fstab里面的dump设置的分区是否备份过
		更多参数man dump.
4. 文件系统的备份：
具体可以参考:
[Centos7xfs文件系统的dump备份](https://jingyan.baidu.com/article/17bd8e5223348f85ab2bb821.html)
5. 非文件系统（文件目录）的备份
-u,level 1~9都不适用
将 /etc 整个目录通过dump进行备份，且含压缩功能（未测试）
		# dump -0j -f /root/etc.dump.bz2 /etc
一般情况下dump下不会使用压缩功能。
测试的时候发现我的Centos7已经没有了dump这个命令，也没有restore这个命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-07.jpg)
这是因为没有安装dump这个命令,使用如下命令可以安装dump和restore命令：
		yum –y install dump*    //装不了就加*号，一般不加
如下:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-10.jpg)
再man dump一下：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-09.jpg)
可以用了。
5. restore简介
和dump配套，可以还原备份。也分好几种情况：
7. 查看dump后备份数据内容
		restore -t [-f dumpfile] [-h]
		-t:查看备份文件中含有什么重要数据
		-f:后面跟想要处理的dump文件
		-h:查看完整备份数据中的inode与文件系统label等信息
如查看/root/boot.dump备份：
		# restore -t -f /root/boot.dump
8. 比较差异并还原整个文件系统
		# restore -C [-f filename] [-D 挂载点]   //比较dump与实际文件
		# restore -r [-f filename]    //还原整个文件系统
		-C：可以将dump数据和实际文件相比较
		-D：与-C搭配可以查出后面接的挂载点与dump内有不同的文件
		-r：将整个文件系统还原的一种操作
如查看差异并还原/root/boot.dump备份：
		# restore -C -f /root/boot.dump
		# restore -r -f /root/boot.dump
		# partprobe   //刷新分区
9. 仅还原部分文件的restore互动操作（互动模式）
		# restore -i [-f filename]
		-i:进入互动模式，可以仅还原部分文件。
使用该命令后会进入文件环境内，可以使用help查看具体命令。使用quit离开。

---
## 4.光盘写入工具及其他常用工具：
1. mkisofs:新建镜像文件
将数据包成一个镜像文件刻入到DVD时就可以这么做。
		# misk [-o 镜像文件] [-rv] [-m file] 带备份文件.. [-V vol] -graft-point isodir=systemdir ...
		-o:后面接想要产生的那个镜像文件名
		-r:通过Rock Ridge产生支持Unix/Linux的文件数据，可记录较多的信息
		-v:显示构建ISO镜像文件的过程
		-m file:排除文件file,file不会备份到镜像文件中
		-V vol:新建Volume
		-graft-point:graft有移植的意思
默认情况下所有要被加到镜像文件的目录都会被放到镜像文件中的根目录。
2. cdrecord:光盘刻录工具
		# cdrecord -scanbuSdev=ATA       //查询刻录机的位置
		# cdrecord -v dev=ATA:x,y,z blank=[fast|all]   //抹除重复读写片
		# cdrecord -v dev=ATA:x,y,z -format
		# cdrecord -v dev=ATA:x,y,z [可用参数功能] file.iso
		-scanbuS:用在扫描磁盘总线并找出可用的刻录机
		-v:显示过程
		dev=ATA:x,y,z：后续x,y,z为系统上刻录机所在的位置，非常重要
		blank=[fast|all]：blank抹除时是快还是慢
		-format:仅针对DVD+RW这种格式的DVD
		[可用参数功能]:写入CD/DVD时可以使用的参数常见的有
			-data:以数据的格式写入而不是以音轨的形式
			speed=X:指定刻录速度。
			-eject:指定刻录完毕后自动退出光盘
			fs=Ym:指定多少缓冲存储器，默认4m,建议8m
			针对DVD的功能有:
			dirveropts=burnfree:打开Buffer Underrun Free模式的写入
			-sao:支持DVD-RW的格式
3. dd:制作文件，备份
dd的功能不只在于制作一个大文件，更重要的是可以备份：
		# dd if="input file" of="output file" bs="block size" count="number"
		if:input file,也可以是设备文件
		of:output file,也可以是设备文件
		bs:规划一个block大小，未指定则是512bytes
		count:多少个bs
tar可以用来备份关键数据，而dd则可以用来备份整块磁盘分区或整块磁盘。
尤其是要将数据写回到文件系统时要考虑原有的文件系统。
4. cpio:
可以备份任何东西，包括设备文件。
但是有个致命缺点：cpio不会主动去找文件来备份，需要配合find使用，还要用到数据流重定向。
		# cpio -ovcB > [file|device]    //备份
		# cpio -ivcdu < [file|device]   //还原
		# cpid -ivct < [file|devic]     //查看
		-o:将数据copy输出到文件或设备上
		-B:让默认的Block增加到10倍（5120bytes）,可以让大文件的存储速度加快
		-i:将数据自文件或设备复制到系统中
		-d:启动新建目录
		-u:自动用新文件将旧文件覆盖
		-t:配合-i可以看出新建文件的内容
		-v:显示存储过程中的文件名
		-c:使用portable format存储
备份：
可以通过（>）将数据重新导向一个新文件
还原，查看：
通过（<）将将备份文件读取进来处理
若要备份到设备文件/dev/st0可以这么写：
备份：find / |cpio-ocvB>/dev/st0
还原：cpio-idvc</dev/st0

---
## 5.vi与vim简介
非常重要，待学待写


---