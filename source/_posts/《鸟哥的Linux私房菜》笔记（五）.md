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
- vi及vim(很重要)

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
		# gzip -v man.config   //压缩，会将源文件删除，显示压缩比
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
		【可用参数功能】:写入CD/DVD时可以使用的参数常见的有
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
1. Linux下的部分命令行文本编辑工具：
Emacs,pico,nano,joe,vim等。
2. 关于vi与vim:
要使用Linux至少得学会一种以上的命令行工具，所有的Linux发行版都会有一套文本编辑器--vi,vim是高级版的vi,vim不但可以用不同的颜色显示文字内容，还能够进行诸如shell脚本，c等程序的编辑功能。
3. 学习vi与vim的原因：
	- 所有的Unix Like系统都会内置vi编辑器，其他文本编辑器则不一定会存在。
	- 很多软件的编辑接口都会主动调用vi。
	- vim具有程序编辑功能，可以主动以字体颜色辨别语法的正确性，方便程序设计。
	- 程序简单，编辑速度相当快。

---
## 6.vi的使用
**1)vi的三种常用模式：**
1. 一般模式：
以vi打开一个文件`# vi **`就直接进入了一般模式（默认模式），此模式下可以使用上下左右移动光标，可以删除字符或者删除整行数据，也可以复制粘贴文件数据，但是，**无法编辑文件内容!**
2. 编辑模式：
一般模式下无法编辑，要等到你按下`i,I,o,O,a,A,r,R`等按钮才会进入编辑模式，屏幕左下方会出现INSERT/REPLACE字样此时才可以进行编辑，按ESC退出编辑模式。
3. 命令行模式：
在一般模式下按`:`,`/`,`?`这三个中任意一个按钮，此时光标会移动到最下面一行，查找数据，**读取保存大量字符，退出vi,显示行号**等操作都是在此模式中完成的。按ESC退回一般模式。

4. 编辑模式与命令行模式无法相互切换。

**2)按键说明(图片来自第四版PDF)：**
1. 使用`# vi 文件名`可以打开或创建一个文件，如果不是新文件的底部会出现这样的文字：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-11.jpg)
表示有13行，37个字符。（这是新文件保存说明有这么多写入）

2. 一般模式下的按键
总表：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-12.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-13.jpg)
常用的按键：
	>hjkl：左下上右移动光标
	>若想进行30次向下移可以输入30j或30下键
	>[Ctrl]+f：向下移动一页，相当于[Page Down]
	>[Ctrl]+b：向上移动一页，相当于[Page Up]
	>0或功能键[Home]：移动到这一行的最前面
	>$或功能键[End]：移动到这一行最后面的字符处
	>G：移动到这个文件的最后一行
	>nG：n为数字，移动到这个文件的第n行
	>gg：移动到这个文件的第一行，相当于1G
	>n[Enter]：n为数字，向下移动n行
	>/word：向下寻找一个名称为word的字符串
	>?word：向上寻找一个名称为word的字符串
	>n：重复前一个查询操作
	>N：方向重复前一个查询操作
	>:n1,n2s/strA/strB/g：n1n2位数字，表示在n1n2行之间寻找A字符串并全部替换为B
	>:1,$s/strA/strB/g：从第一行到最后一行查找A字符串并全部用B替换
	>:1,$s/strA/strB/gc：从第一行到最后一行查找A字符串并全部用B替换,但是替换前会询问
	>x,X：x向后删除一个字符，X向前删除一个字符。
	>nx：连续向后删除n个字符。
	>dd：删除光标所在的那一整行。
	>ndd：n为数字，删除向下n行
	>yy：复制光标所在的那一行
	>nyy：向下复制n行
	>p,P：p为向下一行粘贴已复制的数据，P为向上一行粘贴已复制的数据
	>u：复原（恢复，撤销）上一个操作
	>[Ctrl]+r：重做前一个操作
	>.  ：小数点，重复上一个操作

3. 一般模式到编辑模式的按键：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-14.jpg)
	>i,I：i从目前光标所在处插入，I为在目前所在行第一个非空字符处插入
	>o,O：o从光标下一行插入新的一行，O为在上一行插入新的一行
	>a,A：a从光标下一个字符插入，A从光标所在行最后一个字符处插入
	>r,R：r会替换光标所在的字符一次，R会一直替换。

4. 一般模式到命令行模式的按键：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-15.jpg)
	>:w：将编辑的数据写入到硬盘中
	>:w!：强制写入，即使权限为只读
	>:q：退出，离开vim
	>:wq：保存后离开，:wq!则是保存后离开

**3)vim保存文件、恢复与打开时的警告信息**
在vim一般模式按下[Ctrl]+z时会把vim放到后台执行，这时使用`# ls -al`会发现目录下多了一个隐藏的备份文件（.swap）：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-16.jpg)
如果再次以vim打开此文件（te_issue.txt）则会提示你有一个暂存文件，让你对它进行操作：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-17.jpg)
O：打开为只读
E：强制打开要编辑的文件`te_issue.txt`
R：加载暂存文件
D：删除暂存文件
Q：退出vim
A：放弃此次编辑，与Q差不多
如果vim工作不正常中断，这个文件会被保留，可以继续未完成的操作：
如使用`# kill -9 1%`模拟断电。
暂存文件不会自动删除，需要使用D来手动删除，否则每次打开原文件都会提示。

---
## 7.vim的常用功能

**1)先查看一下系统默认的别名：**
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-18.jpg)
书上的Centos5.x是有一个`alias vi='vim'`的，但是我的Centos7没有，所以可以直接使用`# vim ***`来打开一个文件。

**2)vim的简单使用：**
其实和vi一毛一样，但是文字会有颜色（以`#`开头），而且在右下角多了两个文字说明，前面的是指游标所在的位置（第几行第几个字符），后面的是指当前页面占总页面的屏幕占比：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-19.jpg)
而且vim还多了许多功能。

**3)块选择：**
前面所提到的vi操作都是以行为单位的，如一个文件显示如下：

		192.168.1.1 host1.6666
		192.168.1.2 host2.6666
		192.168.1.3 host3.6666
		192.168.1.4 host4.6666
		192.168.1.5 host5.6666
		192.168.1.6 host6.6666
若想要把每一行的host都复制到后面去，即每一行都是这种格式:`192.168.1.n hostn.6666 hostn`
这时就可以使用块选择了。
块选择的按键:

	v：字符选择，光标经过的地方会反白选择
	V：行选择，光标经过的行反白选择
	[Ctrl]+v/V：块选择，更加精确的选择
	y：将反白的地方复制
	d：将反白的地方删除
	p/P：粘贴
如光标从host1的h到host6的6各个按键的效果：
1. v:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-20.jpg)
2. V:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-21.jpg)
3. [Ctrl]+v/V:
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-22.jpg)

再次点击那个按键或者按esc键就可以退出选择。
**4)多文件编辑：**
主要是为了方便文件与文件之间的复制粘贴。
可以使用`# vim 文件1 文件2...` 这样的格式打开多个文件。
打开文件并水平窗口显示：

		vim -o file1 file2
打开文件并垂直方式显示：

		vim -O file1 file2
如我要打开te_issue.txt和test3-19.txt可以这样写：
		# vim te_issue.txt test3-19.txt
多文件按键：

	:n:编辑下一个文件
	:N:编辑上一个文件
	:files：列出目前这个vim打开的所有文件
如使用`:files`查看刚刚打开的文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-23.jpg)
如果已经打开了一个文件，这时需要打开另一个文件，可以在命令行模式下使用如下命令：

		:open filename
来打开新文件。
具体可以参考：
[Centos7关于几个常见问题的解决方案](Centos7几个常见问题解决方案.md)（第3个问题）
保存关闭所有文件：`:wqa`

**5)多窗口功能：**
如果有两个需要对照着看的文件，使用前面的单窗口多文件方式会很麻烦。
可以使用多窗口功能，在vim命令行模式内输入：

	:vs/vsp ：文件路径/文件名      在新的垂直分屏中打开文件(左右分)
	:sp：文件路径/文件名      在新的水平分屏中打开文件(上下分)
若已经有两个水平窗口了，再执行`：sp filename`则会打开三个水平窗口。
常用按键：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-24.jpg)

		Ctrl+w+方向键——切换到前／下／上／后一个窗格
		Ctrl+w+h/j/k/l ——同上
		Ctrl+ww——依次向后切换到下一个窗格中
第一次`:q`不会退出vim，只会退出多窗口模式。
**6)vim的挑字补全功能：**
一般好用一些的编辑器都具有两个功能：(1)语法检验，(2)挑字补全。
vim的代码检验由颜色高亮来完成了，挑字补齐则可以使用如下快捷键组合：
>[Ctrl]+x-->[Ctrl]+n：通过目前这个正在编辑文件的"内容文字"予以补齐
>[Ctrl]+x-->[Ctrl]+f：以当前目录内的"文件名"作为补齐
>[Ctrl]+x-->[Ctrl]+o：以文件"扩展名"作为补齐（最常用）

下面是书上（第四版）上的一个例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-26.jpg)

---
## 8.vim的环境设置与记录
**1)记录的功能：(~/.viminfo)**
当我们编辑同一个文件时，第二次打开该文件时，光标会在上一次离开的地方。这个记录操作的文件就是`~/.viminfo`
**2)vim环境设置选项：(:set all)**
可以在vim命令行模式下输入`:set all`查看所有vim的环境变量，但是超级多,下面图片只是一部分：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-25.jpg)
下面是一些参数的介绍：
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-27.jpg)
缩排：就是在编辑新的一行时会与上一行第一个非空字符串对齐。
这个整体配置是放在/etc/vimrc下的，但是不建议修改这个文件。
**3)修改环境变量：**
建议修改`~/.vimrc`文件(默认不存在，手动创建)，将希望的值写入。
这是一个大家都在推荐的插件，可以研究研究怎么使用：
<https://github.com/spf13/spf13-vim>
**4)vim常用命令示意图：**
![](http://p5ki4lhmo.bkt.clouddn.com/00011%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A05-28.jpg)

---
## 9.其他使用vim的注意事项
这些问题对于我这样一个还没入门的菜鸟来说显然是不存在的，但是还是记一记好，万一呢。
**1)中文编码的问题：**
需要考虑以下四点：
1. Linux默认支持的语系数据：与/etc/sysconfig/i18n这个文件有关
2. 终端接口（Bash）的语系：与LANG有关，使用`# LANG=...`修改
3. 文件原本的编码
4. 打开终端机的软件

建议使用utf8编码。

**2）DOS与Linux断行字符：**
DOS与Linux的断行方式不同，DOS中断行符号为`^M$`称为CR与LF两个符号，而Linux下仅有LF($)。所以在DOS和Linux之间复制纯文本文件时，一定要转换格式：

	# dos2UNIX [-kn] file [newfile]
	# UNIX2dos [-kn] file [newfile]
	-K:保留原本的mtime格式。
	-n:保留原文件，即最后会有两个文件，一个转换了一个未转换。

**3)语系编码转换：**
将所有文件的编码都转换（iconv）：

	# iconv --list
	# iconv -f 原本的编码 -t 新编码 filename [-o newfile]
	--list:列出iconv支持的语系数据
	-f：从那种编码来（from），后接原本的编码
	-t：到哪种编码去（to），后接新编码
	-o：保留源文件，新文件使用-o后的新文件名。

---
## 10.重要命令总结
**1)文件压缩打包**
`# compress[-rcv] 文件或目录` :一种古老的压缩方式
`# uncompress 文件.Z` :一种古老的解压方式
`# gzip [-cdtv#] 文件名` : 应用比较广的压缩方式(-d:解压)
`# gzip -v man.config` : 压缩，会将源文件删除，显示压缩比
`# gzip -d man.config.gz`  :解压缩，会将.gz删除（不要用gunzip）
`# gzip -9 -c man.config > main.config.gz` :将man.config以最优压缩比压缩，并保留原来的文件
`# zcat 文件.gz` :不解压查看纯文本文件的数据
`# zcat man.config.gz`  :上面的例子
`# bzip2 [-cdkzv#] 文件名` :-z压缩，-d解压，-v显示压缩比，#显示等级
`# bzcat 文件名.bz2` :效果同zcat

`# tar [-j][-z] [cv] [-f 新建的文件名] 被压缩的文件/目录` :打包命令
`# tar -jcv -f filename.tar.bz2 要被压缩的文件或者目录名称` :打包并使用bzip2压缩
`# tar -zpcv -f filename.tar.bg 要被压缩的文件或者目录名称` ：打包并使用gzip压缩,并完整备份权限
`# tar -jcv -f 新建文件名 --exclude=不想加的文件1 [--exclude=...] 待打包压缩的文件或目录` :排除某些目录或文件
`# tar [-j][-z] [-tv] [-f 新建文件名]` :查询压缩包内的文件
`# tar -jtv -f filename.tar.bz2` :查看bzip2压缩包
`# tar -ztv -f filename.tar.gz` ：查看gzip压缩包
`# tar [-j][-z] [xv] [-f 新建文件夹] [-C 目录]` :解压压缩包（-x:解打包或解压缩的功能 -C 目录：用在解压时，指定特定的目录解压可以使用这个参数。)
`# tar -jxv -f filename.tar.bz2 -C 欲解压目录` :解压bzip2压缩包
`# tar -zxv -f filename.tar.gz -C 欲解压目录` :解压gzip压缩包
`# tar -cv -f /dev/st0 /root /home /etc` :将/home,/root,/etc备份到磁带机/dev/st0。

```
//将整个目录一边打包一边在/tmp中解开
# cd /tmp
# tar -cvf - /etc |tar-xvf -

//备份
# dump [-Suvj] [-level] [-f 备份文件] 待备份数据
# dump -W
	-S:仅列出后面的待备份数据要多少空间才能备份完毕
	-u:将这次dump时间记录到/etc/dumpdateS文件中
	-v:将dump过程显示出来
	-j:加入bzip2的支持对数据进行压缩，默认等级为2
	-level：备份等级0-9
	-f:类似tar,后面接产生的文件，可接设备文件名
	-W:列出在/etc/fstab里面的dump设置的分区是否备份过

```
`yum –y install dump*` :安装dump与restore

```
//查看dump后备份数据内容
restore -t [-f dumpfile] [-h]
	-t:查看备份文件中含有什么重要数据
	-f:后面跟想要处理的dump文件
	-h:查看完整备份数据中的inode与文件系统label等信息

//比较差异并还原整个文件系统
# restore -C [-f filename] [-D 挂载点]   //比较dump与实际文件
# restore -r [-f filename]    //还原整个文件系统
	-C：可以将dump数据和实际文件相比较
	-D：与-C搭配可以查出后面接的挂载点与dump内有不同的文件
	-r：将整个文件系统还原的一种操作
```

`# restore -i [-f filename]` :仅还原部分文件的restore互动操作（互动模式）



`# misk [-o 镜像文件] [-rv] [-m file] 带备份文件.. [-V vol] -graft-point isodir=systemdir ...` ：镜像文件刻录
```
光盘刻录：
# cdrecord -scanbuSdev=ATA       //查询刻录机的位置
# cdrecord -v dev=ATA:x,y,z blank=[fast|all]   //抹除重复读写片
# cdrecord -v dev=ATA:x,y,z -format
# cdrecord -v dev=ATA:x,y,z [可用参数功能] file.iso
	-scanbuS:用在扫描磁盘总线并找出可用的刻录机
	-v:显示过程
	dev=ATA:x,y,z：后续x,y,z为系统上刻录机所在的位置，非常重要
	blank=[fast|all]：blank抹除时是快还是慢
	-format:仅针对DVD+RW这种格式的DVD
	【可用参数功能】:写入CD/DVD时可以使用的参数常见的有
		-data:以数据的格式写入而不是以音轨的形式
		speed=X:指定刻录速度。
		-eject:指定刻录完毕后自动退出光盘
		fs=Ym:指定多少缓冲存储器，默认4m,建议8m
		针对DVD的功能有:
		dirveropts=burnfree:打开Buffer Underrun Free模式的写入
		-sao:支持DVD-RW的格式

dd:制作大文件，备份
# dd if="input file" of="output file" bs="block size" count="number"
	if:input file,也可以是设备文件
	of:output file,也可以是设备文件
	bs:规划一个block大小，未指定则是512bytes
	count:多少个bs

cpio备份：需与find结合使用
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

若要备份到设备文件/dev/st0可以这么写：
备份：find / |cpio-ocvB>/dev/st0
还原：cpio-idvc</dev/st0
```


**2）vim及vim内部命令**
`# vi **`，`# vi **` ：打开一个或多个文件

一般模式的按键：
>hjkl：左下上右移动光标
>若想进行30次向下移可以输入30j或30下键
>[Ctrl]+f：向下移动一页，相当于[Page Down]
>[Ctrl]+b：向上移动一页，相当于[Page Up]
>0或功能键[Home]：移动到这一行的最前面
>$或功能键[End]：移动到这一行最后面的字符处
>G：移动到这个文件的最后一行
>nG：n为数字，移动到这个文件的第n行
>gg：移动到这个文件的第一行，相当于1G
>n[Enter]：n为数字，向下移动n行
>/word：向下寻找一个名称为word的字符串
>?word：向上寻找一个名称为word的字符串
>n：重复前一个查询操作
>N：方向重复前一个查询操作
>:n1,n2s/strA/strB/g：n1n2位数字，表示在n1n2行之间寻找A字符串并全部替换为B
>:1,$s/strA/strB/g：从第一行到最后一行查找A字符串并全部用B替换
>:1,$s/strA/strB/gc：从第一行到最后一行查找A字符串并全部用B替换,但是替换前会询问
>x,X：x向后删除一个字符，X向前删除一个字符。
>nx：连续向后删除n个字符。
>dd：删除光标所在的那一整行。
>ndd：n为数字，删除向下n行
>yy：复制光标所在的那一行
>nyy：向下复制n行
>p,P：p为向下一行粘贴已复制的数据，P为向上一行粘贴已复制的数据
>u：复原（恢复，撤销）上一个操作
>[Ctrl]+r：重做前一个操作
>.  ：小数点，重复上一个操作

一般模式到插入模式的按键：
>i,I：i从目前光标所在处插入，I为在目前所在行第一个非空字符处插入
>o,O：o从光标下一行插入新的一行，O为在上一行插入新的一行
>a,A：a从光标下一个字符插入，A从光标所在行最后一个字符处插入
>r,R：r会替换光标所在的字符一次，R会一直替换。

一般模式到命令行模式的按键：
>:w：将编辑的数据写入到硬盘中
>:w!：强制写入，即使权限为只读
>:q：退出，离开vim
>:wq：保存后离开，:wq!则是保存后离开


一般模式块选择按键：

	v：字符选择，光标经过的地方会反白选择
	V：行选择，光标经过的行反白选择
	[Ctrl]+v/V：块选择，更加精确的选择
	y：将反白的地方复制
	d：将反白的地方删除
	p/P：粘贴

多文件编辑命令：

		:n:编辑下一个文件
		:N:编辑上一个文件
		:files：列出目前这个vim打开的所有文件


多窗口命令（按键）：

		Ctrl+w+方向键——切换到前／下／上／后一个窗格
		Ctrl+w+h/j/k/l ——同上
		Ctrl+ww——依次向后切换到下一个窗格中

挑字补齐功能：
>[Ctrl]+x-->[Ctrl]+n：通过目前这个正在编辑文件的"内容文字"予以补齐
>[Ctrl]+x-->[Ctrl]+f：以当前目录内的"文件名"作为补齐
>[Ctrl]+x-->[Ctrl]+o：以文件"扩展名"作为补齐（最常用）


`:set all` ：查看所有环境变量的配置

DOS和Linux格式转换：

		# dos2UNIX [-kn] file [newfile]
		# UNIX2dos [-kn] file [newfile]
		-K:保留原本的mtime格式。
		-n:保留原文件，即最后会有两个文件，一个转换了一个未转换。

语系编码的转换：

		# iconv --list
		# iconv -f 原本的编码 -t 新编码 filename [-o newfile]
		--list:列出iconv支持的语系数据
		-f：从那种编码来（from），后接原本的编码
		-t：到哪种编码去（to），后接新编码
		-o：保留源文件，新文件使用-o后的新文件名。

---