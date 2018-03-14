---
title: 《鸟哥的Linux私房菜》笔记（四）:Linux磁盘与文件系统的管理
date: 2018-03-12 19:28:45
tags: Linux
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第8章:Linux磁盘与文件系统的管理

---
## 1.认识EXT2文件系统
### 1.1 EXT2简介与结构
1. Windows与Linux的操作系统：
windows2000以后使用的是NTFS文件系统，Linux的正规文件系统则为Ext2。
一个可被[挂载](https://baike.baidu.com/item/%E6%8C%82%E8%BD%BD/2366421?fr=aladdin)的数据为一个文件系统而不是分区。
2. Linux的三种块:
	inode：记录了文件的权限和属性，一个文件占用一个inode，同时记录此文件数据塑在的block号码。
	block：记录实际文件的内容，若文件太大时会占用多个block。
	super block(超级块）：记录文件系统的整体信息，包括inode/block的总量，使用量，剩余量，以及文件系统的格式与相关信息等。
3. inode/block的数据访问过程:
1.文件系统先格式化出inode与block的块
2.找到存放文件属性和权限的inode
3.根据该inode的记录的索引找到所有的block
(使用时间久了最好使用碎片整理整理一下block)
4. Linux的Ext2系统：
文件系统一开始就将inode和block规划好了，厨卫重新格式化（或用resizeof命令改变文件系统大小），否则inode与block固定之后就不再变动了。由于放在一个初始的inode与block是放在一起的，所以文件多了之后必须要格式化。
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-1.jpg)
将文件格式化为多个块组（block group）,每个组块中都有独立的inode/block/superblock系统。
下面是关于各个部分的介绍:
<p>（1）：启动扇区boot sector，可以启动装载程序，可以将不同的引导装载程序安装到不同的文件系统。
（2）：block group：组块，每个块组拥有独立的inode/block，一个文件系统只有一个Superblock。
（3）：Data Block：数据块，用来放置文件内容，block大小有1kb,2kb,4kb,容量如下:
<table>
<tr><th>Block大小</th><th>最大单一文件限制</th><th>最大文件系统总容量</th></tr>
<tr><td>1kB</td><td>16GB</td><td>2TB</td></tr>
<tr><td>2kB</td><td>256GB</td><td>8TB</td></tr>
<tr><td>4kB</td><td>2TB</td><td>16TB</td></tr>
</table>
注意:block大小被格式化后就不能被改变，**每个block最多只能放一个文件**，如果一个文件大小>block容量，则会占用多个block，若比block小，则剩下空间浪费（分段存储）。
（4）：Inode Table:存放文件属性和权限等。
	-inode存放的文件数据至少有：
	-文件的访问权限(rwx)
	-文件的所有者与组(ower/group)
	-文件的大小
	-文件创建和状态改变时间
	-最近一次读的时间
	-最近修改的时间
	-文件类型标识
	-文件指向的block号
**inode的大小固定为128B**
我们重点看一下最后一项，“文件指向的block号”
**inode是通过12个直接指针，1个间接指针，1个双间接指针，1个三间接指针来指向block的。**如果文件太大,就会使用间接指针，双间接指针，三间接指针来记录编号。
（5）：Superblock:存放文件系统的基本信息。
**一个文件系统只有一个Superblock**，存放的信息有：
&nbsp;&nbsp;&nbsp;&nbsp;-inode,block的总量
&nbsp;&nbsp;&nbsp;&nbsp;-未使用和已使用的inode,block数量
&nbsp;&nbsp;&nbsp;&nbsp;-inode,block的大小
&nbsp;&nbsp;&nbsp;&nbsp;-文件系统挂载时间，最近写入数据时间，最近检查磁盘时间
&nbsp;&nbsp;&nbsp;&nbsp;-validbit值，文件系统已挂载，则validbit为0，否则为1
每个区段与superblock的信息可以使用dumpe2fs这个命令来查询:
		# dumpe2fs [-bh] 设备文件名
		-b:列出保留为坏道的部分（用不到）
		-h:仅列出superblock的数据，不会列出其他区段内容
（6）：File system Description(文件系统描述):每个块组的开始结束号码
（7）：block bitmap(区块对照表)：标识block是否使用,便于系统快速找到空间来处置文件
（8）：inode bitmap(inode对照表):标识inode是否使用,作用同上

### 1.2 EXT2与目录，EXT3的关系及挂载
1. 与目录树的关系:
目录记录文件名，文件记录数据。
1)**目录**：
	新建一个目录时，ext2会分配一个inode和至少一块block给该目录。
	inode记录目录权限和属性，以及分配的block号。
	block记录目录下的文件名和文件名占用的inode号。
	目录并不只会占一个block也可能会占用多个。
	查看目录的属性:
		# ll -d / /bin /etc....(你想查看属性的目录)
2)**文件**：
	新建一个文件时，ext2会分配一个inode和对应文件大小的N个block块给该文件。
	inode和文件名会同时被记录在目录的block中，以便通过目录访问到该文件。block存放文件内容。
	查看目录内文件占用的inode的号码可以使用"ls -i"命令,如:
		# ls -li
3）**文件读取，查找**
	查找文件时，会先找到文件所在目录，目录的inode对应的block中，
	存放着文件的名称和inode，找到文件名对应的inode,
	然后找到文件inode对应的block，找到文件内容。
	如读取/etc/passwd文件可以这样读取:
		# ll -di / /etc /etc/passwd
2. EXT2与EXT3的区别：
1）**中间数据**：
一般我们将inode table与data block称为数据存放区，其他的例如superblock，block bitmap等的数据是经常变动的，称为中间数据。每次增删改都会影响这三部分数据。
2）**数据不一致的状态**：
一般情况下，都是可以正常运行的，但是出现意外情况导致系统意外中断产生中间数据与实际存放数据不一致的情况。EXT2的解决方法就是将中间数据与实际数据一一对比，非常浪费时间和资源，由此引出了**日志文件系统**。
3）**日志文件系统**:
分为三步工作:
（a）预备:当系统要写入文件时，先在日志记录块中记录某个文件准备要写入的信息。
（b）实际写入:开始写入文件权限与数据，开始更新中间数据。
（c）结束:完成实际数据与中间数据的更新后，记录完成信息。
4）ETX3与ETX2的最大区别就是使用了日志文件系统：
可利用性，数据完整性，速度快以及易于转换。
3. Linux文件系统的操作：
异步处理，将内存中改动过的文件设为dirty，未改动过的设为clean，系统会不定期将内存中标记为dirty的文件写回磁盘，以保证数据的一致性，也可以通过`# syn`来强制写回磁盘。
注意：系统会将常用的文件数据放到主存储区的缓冲区，以加速文件系统的读写。
因此Linux的物理内存最后都会被用光，这是正常的情况，可以加速系统性能。
4. 挂载的意义:
**文件系统要挂载到目录树，才能使用。**
1）什么是挂载?
将文件系统与目录树结合的操作称为挂载。
2）挂载点一定是目录，该目录成为进入该文件系统的的入口
3）可以通过不同的文件的inode号码判断是否为同一个文件。

---
## 2.文件系统的简单操作
1. 磁盘与目录的容量：df,fu
df：列出文件系统的整体磁盘使用量
du：评估文件系统的磁盘使用量(常用于评估目录所占容量)
1）列出文件系统的整体磁盘使用量:**df**
		# df [-ahikHTm] [目录或文件名]
		-a:列出所有的文件系统，包括系统特有的/proc等文件系统
		-k:以KB为单位显示各文件系统
		-m:以MB为单位显示各文件系统
		-h:以人们较易阅读的GB,MB,KB等格式自行显示（常用）
		-H:以十进制M=1000K代替M=1024的进位方式
		-T:连同该分区的文件系统名称也列出
		-i:不用硬盘容量，以inode的数量来显示(常用)
若`# df`后面不加目录或文件名将会列出系统中所有文件系统。
		# df
		# df -h
		# df -ah /etc
`/proc`是Linux系统需要加载的系统数据，而且是挂载在内存中，所以没有任何硬盘空间。显示出的都是0。
2）评估文件系统的磁盘使用量：**du**
		# du [-ahskm] [文件或目录名]
		-a:列出所有文件的目录容量
		-h:艺人们较易读懂的容量格式显示
		-s:列出总量而已，不列出每个各别的目录占用容量
		-S:不包括子目录下的总计，与-s有些差别
		-k:以KB列出容量显示
		-m:以MB列出容量显示
不加文件或目录名则列出当前目录下所有容量，也可以使用通配符代表目录下所有文件（常用）：
		# du     --不加参数显示当前目录
		# du -sm /*       --通配符，常用
与`df`不同，du会到文件系统内部去查找所有的文件数据。结果表明刚安装好的Linux容量最大的是/usr
2. 连接文件：**ln**
	- 硬连接与软连接：
	（1）硬连接（hard link或实际连接）
	realinode-->realblock
	其余的连接文件都是inode-->block-->realinode指向realinode的，就算有文件`ln`到inode上，也会连到realinode。
	删除其中一个连接文件没关系，可以用另一个连接文件访问正确的数据。
	硬连接有限制:不能跨文件系统，不能连接到目录
	（2）软连接（symbolic link或符号连接）
	像一个链表一样连下去，类似于Windows的快捷方式
	realinode-->realblock
	第一个：oneinode-->oneblock-->realinode
	第二个若连到oneinode：twoinode-->twoblock-->oneinode,以此类推。
	若删除了oneinode，访问twoinode则会报"无法打开该文件"。
	软连接的文件名ls时则会有->连接实际文件
	（3）注意:
	硬连接连起来的inode是同一个，上面的只是便于理解，即不会使用新的inode。而软连接是要使用新的inode，说明产生了新文件。的可以通过`# ls -il`查询。
	- ln语法:
			ln [-sf] 源文件 目标文件
			-s:不加，硬连接，加-s则变成软连接
			-f:若目标文件已存在，则直接删除后创建
	软连接测试:
			# ln issue issue_ha
			# ln -s issue issue_sy
			# ls -il issue*
	也可以软连接到目录:
			# ln -s /tmp /666   --此时删除/666中的数据就是把暂存区的数据都删除
	- 关于目录的连接数:
	新建目录的连接数为2，上级目录连接数+1

---
## 3.磁盘的分区、格式化、检验和挂载
1. 在系统中新增一块硬盘的简要过程：
	- 对磁盘进行简要分区（分区）
	- 对该分区格式化，以创建文件系统（创建系统目录）
	- 仔细一点的话，可以对磁盘分区检验（检验）
	- 在Linux中，需要创建挂载点（目录），并将它挂载上来。（挂载）
2. 磁盘分区:**fdisk**
		# fdisk [-l] 设备名称
		-l：输出后面接的设备的所有分区内容，若仅有fdisk -l时会把整个系统内所有能找到的设备分区均列出来。
关于`# fdisk -l`列出的信息可以查看博客:
[fdisk -l 命令详解](https://www.cnblogs.com/Dreama/articles/2106812.html)
使用不加参数不带数字的fdisk可以进入磁盘进行操作:
		# df /        --查找可用磁盘文件名，我的是/dev/sda3(真实环境可能有hda)
		# fdisk /dev/sda     （注意最后的数字去掉了）
会出现这个选项：`Command (m for help): `,输入m可以得到帮助列表，记住几个重要的就行了:
		d delete a partition  --删除一个分区
		n add a new partition  --新增一个分区
		p print the partition table  --在屏幕上打印出分区
		q quit without saving change  --退出，不保存刚刚的操作
		w write table to disk and exit  --写入磁盘并退出
下面是`# fdisk -l`输出的磁盘分区信息
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-2.jpg)
从左往右的意义为:
Device:设备文件名
Boot:是否为开机引导模块
Start,End:表示这个分区在哪个柱面之间，以此决定柱面大小
Blocks:以1k为单位容量
id/system:这个分区的文件系统应该是啥，只是提示


---