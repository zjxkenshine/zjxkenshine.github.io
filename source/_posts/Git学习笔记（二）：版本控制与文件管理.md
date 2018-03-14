---
title: Git学习笔记（二）：版本控制与修改管理
date: 2018-03-07 14:05:21
tags: Git
categories: 版本控制工具

---
## 0.学习要点
- 版本回退
- 工作区和暂存区
- 管理修改
- 撤销修改
- 删除文件

---
## 1.版本回退
1. 简介：
Git可以像玩游戏保存存档一样，每当你觉得文件修改到一定程度的时候，就可以“保存一个快照”，这个快照在Git中被称为commit。一旦你把文件改乱了，或者误删了文件，还可以从最近的一个commit恢复，然后继续工作，而不是把几个月的工作成果全部丢失。
2. 多次修改及查看：
第一次修改内容:
		this is my first git file
		第一次更改
使用：
		$ git add 我的第一个git文件.txt
		$ git commit -m "第一次修改"
		[master 1c899ff] 第一次修改
		1 file changed, 2 insertions(+), 1 deletion(-)
添加第一次修改并提交快照。
第二次修改内容：
		this is my first git file
		第一次更改
		第二次修改
同样使用add,commit -m提交:
		$ git add 我的第一个git文件.txt
		$ git commit -m "第二次修改"
查看修改日志(git log):
		$ git log
		commit 7abc597ea71d8c81dba4434c53385fcde1f9e080
		Author: poppy <1754294529@qq.com>
		Date:   Wed Mar 7 15:36:55 2018 +0800
		
		    第二次修改
		
		commit 1c899ffd2132af803435198909390f03a730091b
		Author: poppy <1754294529@qq.com>
		Date:   Wed Mar 7 15:22:44 2018 +0800
		
		    第一次修改
		
		commit f3b074a94ed4509f49ffc1618f6a6cb47156a0f1
		Author: poppy <1754294529@qq.com>
		Date:   Wed Mar 7 13:59:47 2018 +0800
		
		    create the first git file
如果嫌输出信息太多，可以试试加上`--pretty=oneline`参数
		$ git log --pretty=oneline
		7abc597ea71d8c81dba4434c53385fcde1f9e080 第二次修改
		1c899ffd2132af803435198909390f03a730091b 第一次修改
		f3b074a94ed4509f49ffc1618f6a6cb47156a0f1 create the first git file
类似`7abc...e080`这种的是版本号（commit id），是一个SHA1计算出来的一个非常大的数字，用十六进制表示。
3. 回退到上一个版本
Git必须知道当前版本是哪个版本，在Git中，用`HEAD`表示当前版本，也就是最新的提交`7abc...e080`，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。
要把当前版本回退到上一版本可以使用`git reset`命令。
		$ git reset --hard head^
		HEAD is now at 1c899ff 第一次修改
使用`git log --pretty=oneline`查看当前版本库中的状态:
		$ git log --pretty=oneline
		1c899ffd2132af803435198909390f03a730091b 第一次修改
		f3b074a94ed4509f49ffc1618f6a6cb47156a0f1 create the first git file
第二次修改没有了。
Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是把`HEAD`从指向一个版本指向了另一个版本。然后顺便把工作区的文件更新了。所以你让HEAD指向哪个版本号，你就把当前版本定位在哪。
4. 回到未来
这时再想回到第二次修改的状态，可以找到第二次修改的commit id，这里是`7abc...e080`，然后通过`git reset`命令:
		$ git reset --hard 7abc597ea71d8c81dba4434c53385fcde1f9e080
		HEAD is now at 7abc597 第二次修改
回到未来的版本。
若已经关闭了Git Bash,找不到最新版本号了，可以运行命令`git reflog`，它记录了你的每一次命令：
		$ git reflog
		1c899ff HEAD@{0}: reset: moving to head^
		7abc597 HEAD@{1}: reset: moving to 7abc597ea71d8c81dba4434c53385fcde1f9e080
		1c899ff HEAD@{2}: reset: moving to head^
		7abc597 HEAD@{3}: commit: 第二次修改
		1c899ff HEAD@{4}: commit: 第一次修改
		f3b074a HEAD@{5}: commit (initial): create the first git file
版本号没必要写全，Git会自动去找。当然也不能只写前一两位,可以只写`commit id`的前半部分,如:
		$ git reset --hard 7abc597
		HEAD is now at 7abc597 第二次修改

---
## 2.工作区与暂存区
1. Git和其他版本控制系统如SVN的一个不同之处就是有暂存区的概念。
2. 工作区:
就是在电脑里能看到的目录，比如`learngit`文件夹就是一个工作区：
版本库:
工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。
Git的版本库里存了很多东西，**其中最重要的就是称为stage（或者叫index）的暂存区**，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。分支与`HEAD`暂时不用了解。
![](http://p5ki4lhmo.bkt.clouddn.com/00002git%E5%AD%A6%E4%B9%A02-1.jpg)
3. 暂存区不同时期的变化
我们把文件往Git版本库里添加的时候，是分两步执行的：
第一步是用git add把文件添加进去，实际上就是**把文件修改添加到暂存区**，可以添加多个；
第二步是用git commit提交更改，实际上就是**把暂存区的所有内容提交到当前分支**。
使用`git status`可以查看文件状态。
修改"我的第一个git文件.txt"：
		this is my first git file
		第一次更改
		第二次修改
		第三次修改
未执行`git add`前使用`git status`查看状态:
		$ git status
		On branch master
		Changes not staged for commit:
		  (use "git add <file>..." to update what will be committed)
		  (use "git checkout -- <file>..." to discard changes in working directory)
		
		        modified:   "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		
		no changes added to commit (use "git add" and/or "git commit -a")
说明文件被修改而未被添加。执行`git add`后再使用`git status`查看状态:
		$ git add 我的第一个git文件.txt
		$ git status
		On branch master
		Changes to be committed:
		  (use "git reset HEAD <file>..." to unstage)
		
		        modified:   "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
执行`git commit`一次性把暂存区的所有修改提交到分支：
		$ git commit -m "第三次修改"
		[master ec9e2b9] 第三次修改
		 1 file changed, 2 insertions(+), 1 deletion(-)
然后执行`git status`查看状态:
		$ git status
		On branch master
		nothing to commit, working directory clean
如果没有对工作区做任何修改，那么工作区就是“干净”的，暂存区是空的。

---
## 3.管理修改
1. 为什么Git比其他版本控制系统设计得优秀，因为Git跟踪并管理的是修改，而非文件。增删改文件都算是修改。
2. 验证是管理修改：
继续更改"我的第一个git文件.txt"
		this is my first git file
		第一次更改
		第二次修改
		第三次修改
		第四次修改
执行`$ git commit -m "第四次修改"`，然后执行`git status`得到结果:
		$ git status
		On branch master
		Changes not staged for commit:
		  (use "git add <file>..." to update what will be committed)
		  (use "git checkout -- <file>..." to discard changes in working directory)
		
		        modified:   "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
发现修改并没有提交:
`git commit`只负责把暂存区的修改提交了,但是修改没有添加`git add`到暂存区，也就没有提交。
用`git diff HEAD -- 我的第一个git文件.txt`命令可以查看工作区和版本库里面最新版本的区别:
		$ git diff HEAD -- 我的第一个git文件.txt
		diff --git "a/\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt" "b/\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		index 35eea08..064a7d0 100644
		--- "a/\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		+++ "b/\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		@@ -1,4 +1,5 @@
		 this is my first git file
		 第一次更改
		 第二次修改
		-第三次修改
		\ No newline at end of file
		+第三次修改
		+第四次修改
		\ No newline at end of file
执行`git add,git commit`后再执行`git diff HEAD -- 我的第一个git文件.txt`命令:
		$ git diff HEAD -- 我的第一个git文件.txt
发现没有了一连串的尾巴。
3. 每次修改，如果不`add`到暂存区，那就不会加入到`commit`中。

---
## 4.撤销修改
1. 执行了一次错误的修改:
		this is my first git file
		第一次更改
		第二次修改
		第三次修改
		第四次修改
		错误的修改
2. 若未使用`git add`添加到暂存区，或者使用`git add`添加到暂存区后又做了修改：
命令`git checkout -- file`可以丢弃工作区的修改。执行:
		$ git checkout -- 我的第一个git文件.txt
没有任何尾巴，继续查看工作区文件，发现已经回到了修改前的样子：
		this is my first git file
		第一次更改
		第二次修改
		第三次修改
		第四次修改
命令`git checkout -- file`意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：
(1)一种是文件被修改后还没有被add到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
(2)一种是文件已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。
注意：`git checkout -- file`命令中的--很重要，没有--，就变成了“切换到另一个分支”的命令。
3. 若已经使用`git add`添加到暂存区：
用命令`git reset HEAD file`可以把暂存区的修改撤销掉（unstage），重新放回工作区(使用`git status`时会提醒 `use "git reset HEAD <file>..." to unstage`：
		$ git reset HEAD 我的第一个git文件.txt
		Unstaged changes after reset:
		M       我的第一个git文件.txt
再使用`git status`发现暂存区的修改回退到工作区了:
		$ git status
		On branch master
		Changes not staged for commit:
		  (use "git add <file>..." to update what will be committed)
		  (use "git checkout -- <file>..." to discard changes in working directory)
		
		        modified:   "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		
		no changes added to commit (use "git add" and/or "git commit -a")
这样就和2的情况一样了，再使用`git checkout -- file`撤销工作区的修改。
		$ git checkout -- 我的第一个git文件.txt

---
## 5.删除文件
1. 在git中，**删除也是一种修改操作**。
新建一个file2.txt并add,commit.
		$ git add file2.txt
		$ git commit -m "add new file2"
		[master 3c22783] add new file2
		 1 file changed, 0 insertions(+), 0 deletions(-)
		 create mode 100644 file2.txt
删除文件可以直接在文件管理器右键删除，也可以使用命令行`rm file`:
		$ rm file2.txt
又没有尾巴，但是文件已经删除了,使用`git status`会立即得知那些文件被删除了，并给了两个选项:
		(use "git add/rm <file>..." to update what will be committed)
		(use "git checkout -- <file>..." to discard changes in working directory)
2. 确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`：
		$ git rm file2.txt
		rm 'file2.txt'
		$ git commit -m "delete file2.txt"
		[master 5b3b482] delete file2.txt
		 1 file changed, 0 insertions(+), 0 deletions(-)
		 delete mode 100644 file2.txt
3. 另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
		$ git checkout -- file2.txt
`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以一键还原。

---
## 6.学习总结
1. 命令行:
`git log`查看版本日志
`git log --pretty=oneline`查看修改日志(格式化)
`git reset --hard head^`回到上一个版本
`git reset --hard head~n`回到n个版本以前
`git reset --hard commit_id`回到commit_id所对应的版本
`git reflog`记录每一个命令操作
`git status`查看当前文件状态
`git diff file`查看具体修改了什么内容
`git diff HEAD -- file`命令可以查看工作区和版本库里面最新版本的区别
`git checkout -- file`让这个文件回到最近一次`git commit`或`git add`时的状态
`git reset HEAD file`把暂存区的修改撤销，重新放回工作区
`git rm file`删除文件

2. 学习地址:
<https://www.liaoxuefeng.com/>

---