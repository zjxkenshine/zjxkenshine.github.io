---
title: Git学习笔记（一）：Git简介与配置安装
date: 2018-03-07 11:28:37
tags: Git
categories: 版本控制工具

---
## 0.学习要点
- 了解Git
- 集中式与分布式
- Git的安装及配置
- 简单使用git

---
## 1.Git简介
1. Git是什么
Git是目前世界上最先进的**分布式版本控制系统**（没有之一）。
2. Git的的诞生
2005年，Linux的开发者Linus用C写了一个分布式版本控制系统，用于管理Linux源代码，Git迅速成为最流行的分布式版本控制系统。
2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub。

---
## 2.集中式与分布式
### 2.1 集中式版本控制系统
1. 简介：
集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。（CVS/SVN/ClearCase/VSS等）
2. 示意图:
![](http://p5ki4lhmo.bkt.clouddn.com/00001git%E5%AD%A6%E4%B9%A01-1.jpg)
3. 缺点
集中式版本控制系统最大的毛病就是必须联网才能工作，如果在局域网内还好，带宽够大，速度够快，可如果在互联网上，遇到网速慢的话，可能提交一个10M的文件就需要5分钟。

### 2.2 分布式版本控制系统
1. 简介：
分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。
分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器的作用仅仅是用来**方便“交换”大家的修改**，没有它大家也一样干活，只是交换修改不方便而已。（Git/BitKeeper/Mercurial/Bazaar等）
2. 优点:
和集中式版本控制系统相比，分布式版本控制系统的安全性要高很多，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，随便从其他人那里复制一个就可以了。而集中式版本控制系统的中央服务器要是出了问题，所有人都没法干活了。
3. 示意图:
![](http://p5ki4lhmo.bkt.clouddn.com/00001git%E5%AD%A6%E4%B9%A01-2.jpg)

---
## 3.Git安装(Windows)
1. 安装:
在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads),或者[百度网盘下载](https://pan.baidu.com/s/1kU5OCOB?errno=0&errmsg=Auth%20Login%20Sucess&&bduss=&ssnerror=0&traceid=#list/path=%2Fpub%2Fgit)，然后选择自己的版本下载，按默认选项安装即可。
安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！
![](http://p5ki4lhmo.bkt.clouddn.com/00001git%E5%AD%A6%E4%B9%A01-3.jpg)
2. 设置用户名和初始地址：
		$ git config --global user.name "Your Name"
		$ git config --global user.email "email@example.com"
user.name "Your Name"：用户名;
user.email "email@example.com"：邮箱地址。
因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。
注意`git config`命令的`--global`参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

---
## 4.创建版本库(仓库):
1. 什么是版本库:
版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。
2. 创建版本库:
第一步：选择一个合适的地方，创建一个空目录：
		$ git mkdir learngit
转到learngit目录并显示：
		$ cd learngit
		$ pwd
		/d/GitHub/learngit
`pwd`命令用于显示当前目录,我的目录位于/d/GitHub/learngit。
如果你使用Windows系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文。
不一定必须在空目录下创建Git仓库，选择一个已经有东西的目录也是可以的。
第二步：通过`git init`命令把这个目录变成Git可以管理的仓库：
		$ git init
		Initialized empty Git repository in D:/GitHub/learngit/.git/
瞬间Git就把仓库建好了，而且告诉你是一个空的仓库（empty Git repository）。
当前目录下多了一个.git的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件。
如果你没有看到.git目录，那是因为这个目录默认是隐藏的，用`ls -ah`命令就可以看见。
3. **版本控制系统无法跟踪二进制文件**
所有的版本控制系统，其实只能跟踪文本文件的改动，比如TXT文件，网页，所有的程序代码等等，Git也不例外。版本控制系统可以告诉你每次的改动，比如在第5行加了一个单词“Linux”等。而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只知道图片从100KB改成了120KB。
Microsoft的Word格式是二进制格式，因此，**版本控制系统是没法跟踪Word文件的改动的**。真要使用就以纯文本方式编辑，因为纯文本有编码。
千万不要使用Windows自带的记事本编辑任何文本文件。会遇到很多不可思议的问题。使用NotePad++。把Notepad++的默认编码设置为UTF-8 without BOM即可。

---
## 5.把文件上传到版本库
1. 新建一个文件"我的第一个git文件.txt",内容如下:
		this is my first git file
一定要放到learngit目录下（子目录也行），因为这是一个Git仓库，放到其他地方Git再厉害也找不到这个文件。
2. 把一个文件放到Git仓库只需要两步。
1.第一步:`git add` 添加到仓库
		git add 我的第一个git文件.txt
执行上面的命令，没有任何显示，说明添加成功。
2.第二步：`git commit`把文件提交到仓库
		$ git commit -m "create the first git file"
		[master (root-commit) f3b074a] create the first git file
		 1 file changed, 1 insertion(+)
		 create mode 100644 "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。若只用`git commit`而不用`-m`则会报错。
3. 为什么Git添加文件需要`add`，`commit`一共两步呢？因为`commit`可以一次提交很多文件，所以你可以多次`add`不同的文件，比如：
		$ git add file1.txt
		$ git add file2.txt file3.txt
		$ git commit -m "add 3 files."

---
## 6.学习总结
1. 命令总结:
```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"``` 
设置用户名和密码
`git init` 把这个目录变成Git可以管理的仓库
`git add file` 添加
`git add -A .` 一次添加所有改变的文件（注意最后有个句点）。
`git add -A` 添加所有内容
`git add .` 添加新文件和编辑过的文件不包括删除的文件
`git add -u` 表示添加编辑或者删除的文件，不包括新添加的文件。
`git commit -m "..."` 批量提交(快照)
2. 学习地址:
<https://www.liaoxuefeng.com/>

---