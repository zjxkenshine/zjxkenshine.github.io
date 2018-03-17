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

2. 学习重点
	- 认识文件系统
	- 磁盘的分区，初始化，检测和挂载
	- 如何设置开机挂载及特殊文件挂载
	- 交换分区的构建
	
3. 注：
本章并没有过多测试，因为硬件条件不支持，以理解为主，以后真正用到了再回来重新学习。

---
## 1.认识EXT2文件系统
### 1.1 EXT2简介与结构
1. Windows与Linux的操作系统：
windows2000以后使用的是NTFS文件系统，Linux的正规文件系统则为Ext2。
一个可被[挂载](https://baike.baidu.com/item/%E6%8C%82%E8%BD%BD/2366421?fr=aladdin)的数据为一个文件系统而不是分区。(**文件系统就是磁盘分区的格式**)
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
5. 其它Linux支持的文件系统:
传统文件系统:ext2/minx/MS-DOS/FAT等
日志文件系统:ext3/ReiserFS/Window'sNTFS等
网络文件系统:NFS/SMBFS
[关于文件系统的更多介绍](https://linux.cn/article-6907-1.html)
查看本机支持的文件系统:
		# ls -l /lib/modules/$(umane -r)/kernel/fs
查看系统目前已经加载到内存中支持的文件系统:
		# cat /proc/filesystem
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
		-h:以人们较易读懂的容量格式显示
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
			# ln -s /tmp/666   --此时删除/666中的数据就是把暂存区的数据都删除
	- 关于目录的连接数:
	新建目录的连接数为2，上级目录连接数+1

---
## 3.磁盘的分区、格式化、检验和挂载
1. 在系统中新增一块硬盘的简要过程：
	- 对磁盘进行简要分区（分区）
	- 对该分区格式化，以创建文件系统（创建系统目录）
	- 仔细一点的话，可以对磁盘分区检验（检验）
	- 在Linux中，需要创建挂载点（目录），并将它挂载上来。（挂载）

### 3.1 磁盘分区
1. 磁盘分区:**fdisk**
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
		t 修改系统分区ID（即sda后的数字）
下面是`# fdisk -l`输出的磁盘分区信息
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-2.jpg)
从左往右的意义为:
Device:设备文件名
Boot:是否为开机引导模块
Start,End:表示这个分区在哪个柱面之间，以此决定柱面大小
Blocks:以1k为单位容量
id/system:这个分区的文件系统应该是啥，只是提示
----删除磁盘分区:
在`Command (m for help):`选择d并指定要删除的分区。
2. 分区练习及介绍：
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-4.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-3.jpg)
要先删除原来的主分区再添加。
4个分区以内会让你选择主分区还是扩展分区,若已有扩展分区则会让你选逻辑分区。First Serctor选择开始分区，默认就行；Last Serctor选择结束分区，使用+来表示想选取多大的分区。
若前4个分区都为主分区，则无法分配第五个分区。
若有一个扩展分区，则会自动选择逻辑分区，连按两次[Enter]可以分配所有剩下的空间。
<blockquote>
Linux 规定了主分区（或者扩展分区）占用 1 至 16 号码中的前 4 个号码。以第一个 IDE 硬盘(如果是SCSI硬盘，则为sd)为例说明，主分区（或者扩展分区）占用了 hda1、hda2、hda3、hda4，而逻辑分区占用了 hda5 到 hda16 等 12 个号码。因此，Linux 下面每一个硬盘总共最多有 16 个分区。
因此 hda1- hda4 是主区的意思。 hda5以后是逻辑分区！！
</blockquote>
以下是我磁盘分区的结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-5.jpg)
按q离开则所有操作不保存，按w则写入磁盘后保存。
关于主分区，扩展分区和逻辑分区的详细介绍:
[Linux主分区，扩展分区，逻辑分区的联系和区别](http://blog.csdn.net/qq_24817093/article/details/77743824)

### 3.2 磁盘格式化与检测
1. 磁盘格式化：**mkfs**,**mke2fs**
		# mkfs [-t 文件系统格式] 设备文件名
		-t:可以接受的文件系统，如ext 3,ext 2,vfat等（系统支持才会生效）
		需要指定分区名称（label），block大小，inode和block数量，是否有日志记录等（journal设为done则有）
		这是个综合命令，执行它后会调用正确的文件执行格式化操作。
		如 # mkfs -t etx3 /dev/sda4,系统会调用mkfs.etx3这个命令来格式化。
使用`# mksf[tab][tab]`可以查看mksf中所有的文件，其中有个mksf.vfat可以将分区格式化为Windows可读的vfat格式。
		# mke2fs [-b block 大小] [-i block大小] [-L 卷标] [-cj] 设备分区
		-b:可以设置每个block的大小，1024，2048，4096b三种
		-i:多少容量给予一个inode
		-c:检查磁盘错误，进下达一次-c:进行快速读取检查；下达两次-c:读写检查
		-L:接卷名称(lable），这个lable是有用的
		-j:注入日志变为EXT3（默认是EXT2）
以下是书本上的例子:
		# mke2fs -j -L "vbird_logical" -b 2048 -i 8192 /dev/hdc6
2. 磁盘检验：**fsck,badblocks**
文件系统运行时会有硬盘与内存数据异步的情况发生，所以死机等会造成文件系统错乱，这是就得使用`fsck`了(file system check):
		# fsck [-t 文件系统] [-ACa] 设备名称
		-t：fsck和mkfs一样也是个综合命令，需要指定文件系统，但是随着Linux发展，通常可以不加这个参数。
		-A:依据/etc/fstab的内容，将需要的设备都扫描一遍，
		-a:自动修复检查有问题的扇区，所以不用一直按y
		-c:可以在检测过程中使用一个直方图来显示当前进度
		ETX2和ETX3的额外参数:
		-f:强制进入细化检查
		-D:针对文件系统下的目录进行优化配置
如检验/dev/hdc6:
		# fsck -C -f -t ext3 /dev/hdc6
同样可以使用tabtab查看fsck下有哪些文件。
注意:
只有root用户且文件系统有问题时才能使用这个命令，正常状态下使用此命令会对系统造成危害，执行fsck时，**被检查分区不能被挂载到文件系统上，即需要卸载状态**。
badblocks:
		# badblocks -[svw] 设备名称
		-s:在屏幕上列出进度
		-v:可以在屏幕上看到进度
		-w:使用写入方式来测试（建议不要使用）
使用badblocks检测:
		# badblocks -sv /dev/hdc6

### 3.3 磁盘的挂(卸)载与参数修改
1. 磁盘挂载:
挂载前需要注意的事项:
&nbsp;&nbsp;&nbsp;&nbsp;单一文件系统应该被重复挂载到不同的挂载点中。
&nbsp;&nbsp;&nbsp;&nbsp;单一目录不应该重复挂载多个文件系统。
&nbsp;&nbsp;&nbsp;&nbsp;作为挂载点的目录理论上应该都是空目录才行，若不是空目录，则挂载文件之后，目录下的文件会暂时消失。
		# mount -a
		# mount [-l]
		# mount [-t 文件系统] [-L Label名] [-o 额外选项] \ [-n] 设备文件名 挂载点
		-a:依照配置文件 /etc/fstab的数据将所有未挂载的磁盘都挂载上来
		-l:单纯输入mount会显示目前的挂载信息，加上-l可增列Label名称
		-t:与mkfs类似，可以加上文件系统类型来指定挂载的类型，常见的有：ext2,ext3,vfat,reiserfs,iso9660,nfs,cifs,smbfs
		-n:在默认情况下，系统会将实际挂载情况实时写入/etc/mtab中，以利于其他程序的运行，在单用户情况下使用-n可以不写入。
		-L:系统出了利用设备文件名之外，还可以利用文件系统的卷标名称进行挂载，最好来进行挂载。最好有一个独一无二的卷标（即名称）。
		-o:后面可以加一些挂载时额外加上的参数:
			ro,ro：只读或可读写
			async,sync：同步或异步写入
			auto,noauto：是否mount -a自动载入。
			其他参数man mount吧
挂载EXT2/EXT3系统:
		# mkdir /mnt/hdc6
		# mount /dev/hdc6 /mnt/hdc6
上述代码未加参数，会进行测试挂载，测试通过则会立即使用该文件系统挂载目录。
关于U盘等的挂载可以参考以下博客：（CD,DVD,软盘等用到时再百度）
[linux挂载命令mount及U盘、移动硬盘的挂载](https://www.cnblogs.com/sunshine-cat/p/7922193.html)
可以通过`# mount -l`查看已经挂载文件系统的目录 
2. 重新挂载根目录与挂载不特定目录:
目录树最重要的地方是根目录，所有根目录不能被卸载，如果根目录变为了只读或者挂载参数需要改变时，就要用到重新挂载:
		# mount -o remount,rw,auto/
`remount.xx`是`-o`的参数，非常重要，尤其是进入单用户模式时，根目录通常被挂载为只读，这时候就要重新挂载了。（Centos7修改密码时）
也可以用mount将某个目录挂到另一个目录去（额外挂载一个目录），不是文件系统的挂载方法，如将/home这个目录暂时挂载到/mnt/home下:
		# mkdir /mnt/home
		# mount --bind /home /mnt/home
此时进入/mnt/home就是进入/home的意思，inode 是同一个。
3. 卸载设备文件目录:
		# umount [-fn]设备文件名或挂载点
		-f:强制删除，可用在类似网络文件系统无法卸载的情况
		-n:不更新/etc/mtab情况下卸载。
只有卸载退出，才能退出光盘，U盘，软盘等设备。
如果目前正在文件系统内，如（/media/cdrom）,则不能卸载当前的文件系统，需要离开文件系统挂载点（# cd）。
4. 使用Label卷标名进行挂载的方法
		# dumpe2fs -h /dev/hdc6  --读出超级块内的数据
		# mount -L "卷标名" /mnt/hdc6  --使用卷标名挂载到目录
5. 磁盘参数的修改及主从设备号:
关于major(主设备号)与minor（从设备号）:文件标识设备的方式
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-6.jpg)
其中的8是主设备号，0123是从设备号。Linux内核认识的设备数据就是通过这两个设备号决定的，不同的磁盘文件名的主从设备号都不同。
	- mknod 创建设备
	一般情况下用不到，一般不需要手动创建设备文件，但是如果某些服务被放到特定目录下（chroot），就要使用mknod了:
			# mknod 设备文件名 [bcp] [Major] [Minor]
			b：设置设备名称为外部的外部的存储设备文件      eg：硬盘
			c：设置设备名称为外部输入设备文件        eg：键盘/鼠标
			P：设置设备名称为FIFO文件
			Major：主要设备名称代码
			Minor：次要设备代码
	如创建一个: /dec/sda6 8,6设备
			# mknod /dev/sda6 b 8 6
	创建一个FIFO文件:
			# mknod /tmp/testpipe p  -- 测试之后记得删除，这个文件有意义
	- e2label 修改卷标名
			# e2label 设备名称 新的label名称
	如将/dev/hdc6的卷标名称改为'test':
			# e2label /dev/hdc6 "test"。
	注意:如果你修改了卷标，刚好和另外的有个分区有相同的卷标，系统就无法判断哪个分区是正确的。
	- tune2fs 快速转换，修改卷标等
			# tune2fs [-jlL] 设备代号
			-j：将ext2的文件系统转换为ext3的文件系统
			-l：将超级块内的数据度出来，该功能类似于dumpe2fs  -h的功能
			-L：修改文件系统的卷标，类似于e2label的功能
	- hdparm 启动DMA
	如果硬盘有DMA模式的功能，系统却没有启动它，那么，硬盘的读取性能可能会降低一半以上，就可以使用该命令来启动DMA模式的功能。该命令有很多的高级的参数设置值，所以不建议随便的修改，否则容易造成硬盘崩溃，使用这个命令，最多的就是启动DMA功能，并测试硬盘的访问性能就可以了。
			# hdparm [-icdmXTt] 设备名称
			-i：将系统启动过程中使用的本身的核心的驱动程序来测试硬盘的测试值取出来，但是这些值不一定是正确的
			-d：设置是否启用dma模式，-d1为启动，-d0为取消
---
## 4.开机挂载与镜像文件loop挂载
1. 基本上所有Linux发行版（distribution）在启动系统时都是根据/etc/fstab文件的配置来挂载分区的。开机的时候将/etc/fstab配置好，就不需要每次进入Linux都挂载了。(但是一般都是已经配置好的，除非添加新硬件或分区)
2. 系统挂载的限制:
	- 根目录必须是挂载的，而且必须先于其他挂载点挂载。
	- 其他挂载点必须为自己新建目录，可以任意指定，但是要遵守系统目录规范。
	- 所有挂载点在同一时间内只能挂载一次。
	- 所有分区在同一时间内只能挂载一次。
	- 卸载时必须将工作目录移到挂载之外。
3. 可以使用cat查看/etc/fstab文件下的内容:
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-7.jpg)
从左到右依次是**label名，挂载点，文件系统，文件系统参数，能否被dumpb备份命令作用，是否以fsck检验扇区**。这六项非常重要。（我刚学Linux没几天，还是不动为好）。以下是第四列的各种参数介绍:
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-8.jpg)
利用mount进行挂载时将所有参数写入这个文件就可以实现开机挂载了。
4. /etc/fstab是开机时的配置文件，实际文件系统的挂载是记录到/etc/mtab和proc/mounts这两个文件夹当中的。在改动文件系统挂载时也会同时更新这两个文件，万一/etc/fstab数据有误需要进入单用户模式维护时，记得remount重新挂载根目录，因为默认是只读的无法更改。
5.  特殊设备loop挂载（镜像文件不刻录就挂载使用）
和使用Linux镜像一样，不刻录就是用里面的数据:使用loop挂载
如有一个centos 7.0_x86_64.iso:
		# mount -o loop /root/centos 7.0_x86_64.iso /mnt/centos_dvd
		# ll /mnt/centos_dvd
此时centos 7.0_x86_64.iso的镜像数据可以在/mnt/centos_dvd看到。(/mnt目录的作用就是拿来暂时挂载数据)最后测试完卸载数据:
		# umount /mnt/centos_dvd
6. 创建大型文件并以loop挂载：
Linux有一个很好呀的命令dd(详细介绍，下一篇压缩)，可以用来创建空文件:
		# dd if=/dev/zero of=/home/loopdev bs=1M count=512
		if：input file,输入文件,/dev/zero是会一直输入0的设备
		of：output file,输出设备,将if输入的0写入到后面接的文件中
		bs：每个block大小，和文件系统的block一样
		count：总共有几个block。
		这里一共是512M的文件
格式化大文件:
		# mkfs -t ext3 /home/loopdev
由于不是正常设备会提示，选择是就行。最后是挂载:
		# mount -o loop /home/loopdev /media/cdrom/

---
## 5.内存交换空间的构建
1. 关于swap，内存交换空间:
安装时一定需要两个分区，一个是根目录，另一个是swap(内存交换空间)，swap的功能是用来应付物理内存不足的情况下所造成的内存扩展记录的功能。
个人用可以不配置（主内存1个G以上）,但是对于服务器一定要有一个交换空间。
2. 使用物理分区构建内存交换空间:
	- 分区:
			# fdisk /dev/hdc
			进入Command环境，新建分区n,t修改系统ID(假设为7),w写入保存
			# partprobe
			让内核更新分区表，很重要的操作
	- 构建swap格式:
			# mkswp /dev/hdc7
	- 查看与加载:
			# free   
			# swapon /dev/hdc7
			# free
			# swapon -s    --列出当前使用swap分区的设备
3. 使用文件构建:
	- 使用dd新建一个128MB的文件
			# dd if=/dev/zero of=/tmp/swap bs=1M count=128
			# ll -h /tmp/swap     --/tmp是存放暂时文件的地方，不定期清一下
	- 使用mkswap将此文件格式化:
			# mkswap /tmp/swap
	- 使用swapon启动/tmp/swap
			# free
			# swapon /tmp/swap
			# free
			# swapon -s
	- 使用swapoff关闭swap file
			# swapoff /tmp/swap
			# swapoff /dev/hdc7 (硬盘构建的)
			# free     --显示当前系统未使用的和已使用的内存数目，还可以显示被内核使用的内存缓冲区
4. swap在使用上的限制:
最多不能超过32个swap，swap的总容量最大不能超过64GB，内核2.4.10版本以前单一swap不能超过2GB（我的是3.10.0，可以使用`# uname -r`查看）

---
## 6.文件系统的特殊查看与操作
1. boot sector与superblock的关系:
	- block为1Kb时:
	boot sector和superblock是分开存储的。
	- block大于1kb(如2k,4k)时:
	为了节省空间第一个文件内就同时含有boot sector和superblock。
2. 磁盘空间的浪费问题:
文件系统会占用一部分磁盘空间。
查询某个目录的所有空间:du,如果加上-s则会以不同规范找出文件系统所消耗的空间。
		# du -sb /etc  --以byte分析
		# du -sm /etc  --以block分析，单位是KB
两者之差就是文件系统的此时的耗费。
3. 利用GUN的parted进行分区：
fdisk无法支持到高于2TB以上的分区，此时就需要parted来处理了。
以下是网上的使用parted分区的大致步骤（以sdb盘为例）：
![](http://p5ki4lhmo.bkt.clouddn.com/00008%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A04-9.jpg)
parted分区的语法:
		# parted [设备] [命令[参数]]   --创建分区
		命令列表:
		新增分区:mkpart [primary|logical|extended] [ext3|vfat] 开始 结束
		分区表:print
		删除分区:rm [partition]
下面是网上的一个例子:
		[root@Candy ~]# parted /dev/sdb
		GNU Parted 2.1
		使用 /dev/sdb
		Welcome to GNU Parted! Type 'help' to view a list of commands.
		(parted) mklabel
		新的磁盘标签类型？ gpt
		(parted) mkpart
		分区名称？  []?
		文件系统类型？  [ext2]? xfs
		起始点？ 0T
		结束点？ 4T
		(parted) p
		Model: VMware, VMware Virtual S (scsi)
		Disk /dev/sdb: 4398GB
		Sector size (logical/physical): 512B/512B
		Partition Table: gpt
		Number  Start   End     Size    File system  Name  标志
		 1      1049kB  4398GB  4398GB
		(parted)set 1 lvm on
		(parted)p
		Model: VMware, VMware Virtual S (scsi)
		Disk /dev/sdb: 4398GB
		Sector size (logical/physical): 512B/512B
		Partition Table: gpt
		Number  Start   End     Size    File system  Name  标志
		1      1049kB  4398GB  4398GB                      lvm
		(parted)q
		[root@Candy ~]# ls /dev/sd*
		/dev/sda  /dev/sda1  /dev/sda2  /dev/sdb  /dev/sdb1

---
## 7.重要命令总结
`# dumpe2fs [-bh] 设备文件名`：查看每个区段与superblock的信息（-b:用不到-h:仅列出superblock的数据）
`# ll -d / /bin /etc....(你想查看属性的目录)`：查看指定目录下的属性
`# ll -di / /etc /etc/passwd`：查看某一文件的属性
`# ls -l /lib/modules/$(umane -r)/kernel/fs`：查看本机支持的文件系统
`# cat /proc/filesystem`：查看系统目前已经加载到内存中支持的文件系统

`# df [-ahikHTm] [目录或文件名]`：列出文件系统的整体磁盘使用量（-a列出所有文件-h以人们易读的容量显示-i以inode节点数显示）
`# du [-ahskm] [文件或目录名]`：评估文件系统的磁盘使用量（-a-h参数同df）
`# du `：不加参数显示当前目录
`# du -sm /* ` ：通配符，常用
**与df不同，du会到文件系统内部去查找所有的文件数据。**

`ln [-sf] 源文件 目标文件`：文件连接（-s:不加，硬连接，加-s则变成软连接-f:若目标文件已存在，则直接删除后创建）
`# ln issue issue_ha`：硬连接到文件issue_ha
`# ln -s /tmp/666`：软连接到目录/tmp/666

`# fdisk [-l] 设备名称`：-l：输出后面接的设备的所有分区内容，若仅有fdisk -l时会把整个系统内所有能找到的设备分区均列出来。
`# df /`：查找可用磁盘文件名。
`# fdisk /dev/sda `：进入磁盘分区环境（可用磁盘文件名的最后数字去掉了）

`# mkfs [-t 文件系统格式] 设备文件名`：磁盘格式化的简单方式
`# mke2fs [-b block 大小] [-i block大小] [-L 卷标] [-cj] 设备分区`：麻烦的磁盘格式化方法(-c:检查磁盘错误，进下达一次-c:进行快速读取检查；下达两次-c:读写检查-j:注入日志变为EXT3（默认是EXT2）)
`# mke2fs -j -L "vbird_logical" -b 2048 -i 8192 /dev/hdc6`：磁盘格式化的例子

`# fsck [-t 文件系统] [-ACa] 设备名称`：检测分区（-A:依据/etc/fstab的内容，将需要的设备都扫描一遍-a:自动修复检查有问题的扇区-c:可以在检测过程中使用一个直方图来显示当前进度。ETX2和ETX3有额外参数:-f强制-D优化）
`# badblocks -[svw] 设备名称`：使用block检测（-s:在屏幕上列进度-v:进度显示）

`# mount -a`：挂载全部
`# mount [-l]`：挂载列表，-l会显示卷名
`# mount [-t 文件系统] [-L Label名] [-o 额外选项] \ [-n] 设备文件名 挂载点` ：挂载文件系统到目录上的一般写法
```
# mkdir /mnt/hdc6
# mount /dev/hdc6 /mnt/hdc6
```
简单挂载EXT2/EXT3系统
`# mount -o remount,rw,auto/`：重新挂载根目录
`# mount --bind /home /mnt/home`：将一个目录挂载到另一个目录
`# umount [-fn]设备文件名或挂载点`：卸载文件系统（f:强制删除，可用在类似网络文件系统无法卸载的情况-n:不更新/etc/mtab情况下卸载。）
`# dumpe2fs -h /dev/hdc6` ：读出超级块内的数据
`# mount -L "卷标名" /mnt/hdc6` ：使用卷标名挂载到目录

`# mknod 设备文件名 [bcp] [Major] [Minor]`：一般情况下用不到，一般不需要手动创建设备文件，但是如果某些服务被放到特定目录下（chroot），就要使用mknod了（Major主设备号,Minor从设备号）
`# mknod /dev/sda6 b 8 6`：创建一个: /dec/sda6 8,6设备（b外部存储设备）
`# mknod /tmp/testpipe p`：创建一个FIFO文件（p:FIFO文件）
`# e2label 设备名称 新的label名称`：修改卷标名
`# e2label /dev/hdc6 "test"`：将/dev/hdc6的卷标名称改为'test'
`# tune2fs [-jlL] 设备代号`：快速修改，修改卷标等（-j：将ext2的文件系统转换为ext3的文件系统-l：将超级块内的数据度出来-L：修改文件系统的卷标）
`# hdparm [-icdmXTt] 设备名称`：操作DMA,(-i:将系统启动过程中使用的本身的核心的驱动程序来测试硬盘的测试值取出来，但是这些值不一定是正确的-d：设置是否启用dma模式，-d1为启动，-d0为取消)

`# mount -o loop /root/centos 7.0_x86_64.iso /mnt/centos_dvd`：loop挂载镜像文件

`# dd if=/dev/zero of=/home/loopdev bs=1M count=512`：使用dd工具创建出512M的文件

`# fdisk /dev/hdc`：进入Command环境，新建分区n,t修改系统ID(假设为7),w写入保存
`# partprobe`：让内核更新分区表，很重要的操作
`# mkswp /dev/hdc7`：构建swap格式
`# free `：显示当前系统未使用的和已使用的内存数目，还可以显示被内核使用的内存缓冲区
`# swapon /dev/hdc7`：加载交换区
`# swapon -s`：列出当前使用swap分区的设备
`# swapoff /tmp/swap`：删除交换区

```
# du -sb /etc  --以byte分析
# du -sm /etc  --以block分析，单位是KB
分析文件系统的消耗
```

创建大分区（GUN的工具parted）：

		# parted [设备] [命令[参数]]   
		命令列表:
		新增分区:mkpart [primary|logical|extended] [ext3|vfat] 开始 结束
		分区表:print
		删除分区:rm [partition]

---
