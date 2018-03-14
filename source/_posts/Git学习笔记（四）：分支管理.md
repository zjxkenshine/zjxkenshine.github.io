---
title: Git学习笔记（四）：分支管理
date: 2018-03-07 20:08:53
tags: Git
categories: 版本控制工具

---
## 0.学习要点
- 什么是分支
- 创建与合并分支
- 解决冲突
- 多人协作开发

---
## 1.分支简介
你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样，既安全，又不影响别人工作。

---
## 2.创建与合并分支
1. 每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。关于分支的具体讲解可以点击[这里](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000)。
2. 创建与切换分支:
		$ git checkout -b dev
		M       "\346\210\221\347\232\204\347\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt"
		Switched to a new branch 'dev'
`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：
		$ git branch dev
		$ git checkout dev
然后用`git branch`命令查看当前分支：
		$ git branch
		* dev
		  master
`git branch`命令会列出所有分支，当前分支前面会标一个*号。
然后，就可以在dev分支上正常提交。
3. 提交到分支:
新建一个文件gitBranchFile.txt，内容如下:
		这是分支文件
并在我的第一个git文件.txt下添加一行:
		分支的修改
提交：
		$ git add -A
		$ git commit -m "分支测试"
		[dev bd5dbdf] 分支测试
		 2 files changed, 4 insertions(+), 1 deletion(-)
		 create mode 100644 gitBranchFile.txt
4. 切换并合并分支：
使用`git checkout`切换到`master`分支:
		$ git checkout master
		Switched to branch 'master'
		Your branch is up-to-date with 'learngit/master'.
这时发现新添加的文件与在我的第一个git文件下的修改都不见了。
再把`dev`分支的工作成果合并到`master`分支上：
		 $ git merge dev
		Updating 5b3b482..bd5dbdf
		Fast-forward
		 gitBranchFile.txt                                                     | 1 +
		 ...47\254\254\344\270\200\344\270\252git\346\226\207\344\273\266.txt" | 4 +++-
		 2 files changed, 4 insertions(+), 1 deletion(-)
		 create mode 100644 gitBranchFile.txt
`git merge`命令用于合并指定分支到当前分支。合并后在看learngit中的内容，发现和提交到`dev`上的是一样的。
`Fast-forward`信息，Git告诉我们，这次合并是“快进模式”，也就是直接把`master`指向`dev`的当前提交，所以合并速度非常快。当然，也不是每次合并都能`Fast-forward`，后面有其他的合并方式。
5. 删除分支：
		$ git branch -d dev
		Deleted branch dev (was bd5dbdf).
再使用`git branch`查看分支:
		$ git branch
		* master
发现只剩下master分支了。

---
## 3.解决冲突
1. 在a分支修改:
新建并切换到a分支：
		$ git checkout -b a
		M       gitBranchFile.txt
		Switched to a new branch 'a'
修改gitBranchFile.txt的内容如下:
		这是分支文件
		a分支的修改
添加提交`git add/commit`（代码略）。
2. 在master分支修改：
切换回master分支:
		$ git checkout master
		Switched to branch 'master'
		Your branch is ahead of 'learngit/master' by 1 commit.
		  (use "git push" to publish your local commits)
Git还会自动提示我们当前`maste`分支比远程的`master`分支要超前1个提交。
继续修改gitBranchFile.txt如下:
		这是分支文件
		master主分支的修改
添加并提交`git add/commit`。
这样`master`与`a`分支都有各自不同的提交。
3. 产生冲突:
这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突：
		$ git merge a
		Auto-merging gitBranchFile.txt
		CONFLICT (content): Merge conflict in gitBranchFile.txt
		Automatic merge failed; fix conflicts and then commit the result.
Git告诉我们，gitBranchFile.txt文件存在冲突，必须**手动解决冲突**后再提交。`git status`也可以告诉我们冲突的文件：
		$ git status
		On branch master
		Your branch is ahead of 'learngit/master' by 2 commits.
		  (use "git push" to publish your local commits)
		You have unmerged paths.
		  (fix conflicts and run "git commit")
		
		Unmerged paths:
		  (use "git add <file>..." to mark resolution)
		
		        both modified:   gitBranchFile.txt
		
		no changes added to commit (use "git add" and/or "git commit -a")
再查看gitBranchFile.txt文件，内容变成了这样:
		这是分支文件
		<<<<<<< HEAD
		master主分支的修改
		=======
		a分支的修改
		>>>>>>> a
Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容。
4. 解决冲突:
修改gitBranchFile.txt文件如下:
		这是分支文件
		master主分支的修改
再提交:
		$ git add -A
		$ git commit -m "解决冲突"
		[master 7f23e9a] 解决冲突
再使用`git status`查看状态：
		$ git status
		On branch master
		Your branch is ahead of 'learngit/master' by 4 commits.
		  (use "git push" to publish your local commits)
		nothing to commit, working directory clean
用带参数的`git log`也可以看到分支的合并情况（用`git log --graph`命令可以看到分支合并图）：
		$ git log --graph --pretty=oneline --abbrev-commit
		*   7f23e9a 解决冲突
		|\
		| * 2cbbe5e a分支的修改
		* | d80455d master的修改
		|/
		* bd5dbdf 分支测试
		* 5b3b482 delete file2.txt
		* 3c22783 add new file2
		* dbb03c7 第四次修改
		* ec9e2b9 第三次修改
		* 7abc597 第二次修改
		* 1c899ff 第一次修改
		* f3b074a create the first git file
最后，删除a分支:
		$ git branch -d a
		Deleted branch a (was 2cbbe5e).

---
## 4.分支管理策略
1. 关于分支合并：
合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。
如果要强制禁用`Fast forward`模式，Git就会在`merge`时生成一个新的`commit`，这样，从分支历史上就可以看出分支信息。使用`git merge --no-ff`。
2. 测试`git merge --no-ff`：
创建并切换到b分支:
		$ git checkout -b b
		M       gitBranchFile.txt
		Switched to a new branch 'b'
在gitBranchFile.txt下添加:
		master的又一次修改
添加并提交:
		$ git add -A
		$ git commit -m "测试--no-ff"
		[b 2ebb903] 测试--no-ff
		 1 file changed, 1 insertion(+), 4 deletions(-)
换回`master`分支：
		$ git checkout master
		Switched to branch 'master'
		Your branch is ahead of 'learngit/master' by 4 commits.
		  (use "git push" to publish your local commits)
使用`git merge`合并，请注意`--no-ff`参数，表示禁用`Fast forward`：
		$ git merge --no-ff -m "不用ff的合并" b
		Merge made by the 'recursive' strategy.
		 gitBranchFile.txt | 5 +----
		 1 file changed, 1 insertion(+), 4 deletions(-)
因为本次合并要创建一个新的`commit`，所以加上`-m`参数，把commit描述写进去。
再使用`git log`看看分支历史（`git log --graph --pretty=oneline --abbrev-commit`）:
		$ git log --graph --pretty=oneline --abbrev-commit
		*   a59aa3f 不用ff的合并
		|\
		| * 2ebb903 测试--no-ff
		|/
		*   7f23e9a 解决冲突
		|\
		| * 2cbbe5e a分支的修改
		* | d80455d master的修改
		|/
		* bd5dbdf 分支测试
		* 5b3b482 delete file2.txt
		* 3c22783 add new file2
		* dbb03c7 第四次修改
		* ec9e2b9 第三次修改
		* 7abc597 第二次修改
		* 1c899ff 第一次修改
		* f3b074a create the first git file
最后删除`b`分支:
		$ git branch -d b
		Deleted branch b (was 2ebb903).
3. 分支管理基本原则
1.master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
2.干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
3.你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
4.团队合作的分支看起来就像这样：
![分支](http://p5ki4lhmo.bkt.clouddn.com/00004git%E5%AD%A6%E4%B9%A04-1.jpg)

---
## 5.BUG分支
1. 使用情景:
软件开发中，bug就像家常便饭一样。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。
当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支`issue-101`来修复它，但是，当前正在dev上进行的工作还没有提交。
并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug。
2. 创建新分支修改bug
正在`dev`上进行工作：
		$ git checkout -b dev
		Switched to a new branch 'dev'
修改gitBranchFile.txt代码如下:
		这是分支文件
		master主分支的修改
		dev工作开始
但是没有添加提交，暂时也无法add，commit。这时忽然需要修改bug。可以使用`git stash`把当前工作现场“储藏”起来，等以后恢复现场后继续工作:
		$ git stash
		Saved working directory and index state WIP on div: a59aa3f 不用ff的合并
		HEAD is now at a59aa3f 不用ff的合并
现在用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。
需要在哪个分支上修复，就从哪个创建临时分支，假设为master分支:
		$ git checkout master
		Switched to branch 'master'
		Your branch is ahead of 'learngit/master' by 6 commits.
		  (use "git push" to publish your local commits)
		$ git checkout -b issue-101
		Switched to a new branch 'issue-101'
假设需要在`master`分支的gitBranchFile.txt的第二行添加上：
		bug修复代码
然后添加提交：
		$ git add gitBranchFile.txt
		$ git commit -m "bug修复"
		[issue-101 2a8a8ec] bug修复
		 1 file changed, 1 insertion(+)
修复完成后，切换到`master`分支，合并并删除`issue-101`分支：
		$ git checkout master
		Switched to branch 'master'
		Your branch is ahead of 'learngit/master' by 6 commits.
		  (use "git push" to publish your local commits)
		$ git merge --no-ff -m "bug修复分支合并" issue-101
		Merge made by the 'recursive' strategy.
		 gitBranchFile.txt | 1 +
		 1 file changed, 1 insertion(+)
		$ git branch -d issue-101
		Deleted branch issue-101 (was 2a8a8ec).
此时master中的代码变成了:
		这是分支文件
		bug修复代码
		master主分支的修改
		master的又一次修改
此时使用`git log --graph --pretty=oneline --abbrev-commit`查看历史：
		$ git log --graph --pretty=oneline --abbrev-commit
		*   786ba47 bug修复分支合并
		|\
		| * 2a8a8ec bug修复
		|/
		*   a59aa3f 不用ff的合并
		|\
		| * 2ebb903 测试--no-ff
		|/
		*   7f23e9a 解决冲突
		|\
		| * 2cbbe5e a分支的修改
		* | d80455d master的修改
		|/
		* bd5dbdf 分支测试
		* 5b3b482 delete file2.txt
		* 3c22783 add new file2
		* dbb03c7 第四次修改
		* ec9e2b9 第三次修改
		* 7abc597 第二次修改
		* 1c899ff 第一次修改
		* f3b074a create the first git file
3. bug修复完后继续未完成的工作:
使用`git checkout dev`切换到`dev`分支并使用`git status`查看状态
		$ git checkout dev
		Switched to branch 'dev'
		$ get status
		On branch dev
		nothing to commit, working directory clean
查看gitBranchFile.txt变成了这样:
		这是分支文件
		master主分支的修改
		master的又一次修改
说明分支是干净的，刚才的工作现场没有保存。用`git stash list`命令查看：
		$ git stash list
		stash@{0}: WIP on dev: a59aa3f 不用ff的合并
工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：
第一种`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；
另一种方式是用`git stash pop`，恢复的同时把stash内容也删了。
		$ git stash pop
		On branch dev
		Changes not staged for commit:
		  (use "git add <file>..." to update what will be committed)
		  (use "git checkout -- <file>..." to discard changes in working directory)
		
		        modified:   gitBranchFile.txt
		
		no changes added to commit (use "git add" and/or "git commit -a")
		Dropped refs/stash@{0} (e43dba357ac81074a485d6bbfccb4ca58f41c10d)
再用`git stash list`查看，就看不到任何stash内容了：
		$ git stash list
你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：
		$ git stash apply stash@{0}
修改`dev`中的gitBranchFile.txt文件,在最后一行加上:
		dev工作完成
添加，提交:
		$ git add -A
		$ git commit -m "git 工作完成"
		[div 3a86388] git 工作完成
		 1 file changed, 2 insertions(+), 1 deletion(-)
切换回`master`主分支:
		$ git checkout master
		Switched to branch 'master'
		Your branch is ahead of 'learngit/master' by 8 commits.
		  (use "git push" to publish your local commits)
合并分支：
		$ git merge --no-ff -m "合并bug与工作" dev
		Auto-merging gitBranchFile.txt
		Merge made by the 'recursive' strategy.
		 gitBranchFile.txt | 3 ++-
		 1 file changed, 2 insertions(+), 1 deletion(-)
发现并未产生冲突，查看gitBranchFile.txt中的代码:
		这是分支文件
		bug修复代码
		master主分支的修改
		dev工作开始
		dev工作完成
若产生冲突需要手动解决冲突。最后关闭`dev`分支：
		$ git branch -d div
		Deleted branch div (was 3a86388).
使用`git log --graph --pretty=oneline --abbrev-commit`查看分支历史：
		$ git log --graph --pretty=oneline --abbrev-commit
		*   97d5cda 合并bug与工作
		|\
		| * 3a86388 git 工作完成
		* |   786ba47 bug修复分支合并
		|\ \
		| |/
		|/|
		| * 2a8a8ec bug修复
		|/
		*   a59aa3f 不用ff的合并
		|\
		| * 2ebb903 测试--no-ff
		|/
		*   7f23e9a 解决冲突
		|\
		| * 2cbbe5e a分支的修改
		* | d80455d master的修改
		|/
		* bd5dbdf 分支测试
		* 5b3b482 delete file2.txt
		* 3c22783 add new file2
		* dbb03c7 第四次修改
		* ec9e2b9 第三次修改

---
## 6.feature分支
1. 使用情况：
添加一个新功能时，不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。
2. 模拟开发
新建一个分支并转入:
		$ git branch -b dev
		$ git checkout -b feature
		Switched to a new branch 'feature'
在gitBushFile.txt下加一行代表新功能开发完毕：
		feature新功能
添加并提交:
		$ git add -A
		$ git commit -m "新功能"
		[feature b7d266d] 新功能
		 1 file changed, 2 insertions(+), 1 deletion(-)
然后你转到`dev`分支准备执行合并，但是这是忽然说这个新功能不要了！
那么销毁feature分支：
若为转到其他分支执行销毁:
		$ git branch -d feature
		error: Cannot delete the branch 'feature' which you are currently on.
若转到了`dev`执行销毁:
		$ git checkout dev
		Switched to branch 'dev'
		$ git branch -d feature
		error: The branch 'feature' is not fully merged.
		If you are sure you want to delete it, run 'git branch -D feature'.
销毁失败。Git友情提醒，`feature`分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用命令`git branch -D feature`。
下面强制删除:
		$ git branch -D feature
		Deleted branch feature (was b7d266d).
删除成功!

---
## 7.多人协作
1. 查看远程库：
从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`。
使用`git remote`可以查看远程库信息（remote:远程的，遥远的）：
		$ git remote
		learngit
或者使用`git remote -v`查看详细信息（fetch：取得，得到）：
		$ git remote -v
		learngit        git@github.com:zjxkenshine/learngit.git (fetch)
		learngit        git@github.com:zjxkenshine/learngit.git (push)
上面显示了可以抓取和推送的`learngit`的地址。如果没有推送权限，就看不到push的地址。
可以用`git remote rename` 命令修改某个远程仓库在本地的简称，比如想把 `learngit` 改成 `learn`，可以这么运行：
		$ git remote rename learngit learn
		$ git remote
		learn
碰到远端仓库服务器迁移，或者原来的克隆镜像不再使用，又或者某个参与者不再贡献代码，那么需要移除对应的远端仓库，可以运行 `git remote rm` 命令：
		$ git remote rm learn
		$ git remote
此时远端仓库已经空了。
2. 推送分支：
推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：
		$ git push origin master
如果要推送其他分支，比如`dev`，就改成：
		$ git push origin dev
但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？(分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！)
	>`master`分支是主分支，因此要时刻与远程同步；
	>`dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
	>`bug`分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
	>`feature`分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。
3. 抓取分支（以下代码是老师的代码）:
多人协作时，大家都会往`master`和`dev`分支上推送各自的修改。
模拟一个你的小伙伴，可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆（尾巴已经省略）：
		$ git clone git@github.com:yourusername/learngit.git
当你的小伙伴从远程库`clone`时，默认情况下，你的小伙伴只能看到本地的`master`分支。可以用`git branch`命令看看：
		$ git branch
		* master
现在，你的小伙伴要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，于是他用这个命令创建本地`dev`分支：
		$ git checkout -b dev origin/dev
现在，他就可以在`dev`继续修改，然后，时不时地把`dev`分支`push`到远程（尾巴都已经省略）：
		$ git commit -m "add /usr/bin/env"
		$ git push origin dev
你的小伙伴已经向origin/dev分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送：
		$ git add hello.py 
		$ git commit -m "add coding: utf-8"
		[dev bd6ae48] add coding: utf-8
		 1 file changed, 1 insertion(+)
		$ git push origin dev
		To git@github.com:michaelliao/learngit.git
		 ! [rejected]        dev -> dev (non-fast-forward)
		error: failed to push some refs to 'git@github.com:michaelliao/learngit.git'
		hint: Updates were rejected because the tip of your current branch is behind
		hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
		hint: before pushing again.
		hint: See the 'Note about fast-forwards' in 'git push --help' for details.
推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用`git pull`把最新的提交从`origin/dev`抓下来，然后，在本地合并，解决冲突，再推送：
		$ git pull
		remote: Counting objects: 5, done.
		remote: Compressing objects: 100% (2/2), done.
		remote: Total 3 (delta 0), reused 3 (delta 0)
		Unpacking objects: 100% (3/3), done.
		From github.com:michaelliao/learngit
		   fc38031..291bea8  dev        -> origin/dev
		There is no tracking information for the current branch.
		Please specify which branch you want to merge with.
		See git-pull(1) for details
		
		    git pull <remote> <branch>
		
		If you wish to set tracking information for this branch you can do so with:
		
		    git branch --set-upstream dev origin/<branch>
`git pull`也失败了，原因是没有指定本地`dev`分支与远程`origin/dev`分支的连接，根据提示，设置`dev`和`origin/dev`的链接：
		$ git branch --set-upstream dev origin/dev
		Branch dev set up to track remote branch dev from origin.
再`pull`：
		$ git pull
		Auto-merging hello.py
		CONFLICT (content): Merge conflict in hello.py
		Automatic merge failed; fix conflicts and then commit the result.
这回`git pull`成功，但是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再`push`：
		$ git commit -m "merge & fix hello.py"
		[dev adca45d] merge & fix hello.py
		$ git push origin dev
		Counting objects: 10, done.
		Delta compression using up to 4 threads.
		Compressing objects: 100% (5/5), done.
		Writing objects: 100% (6/6), 747 bytes, done.
		Total 6 (delta 0), reused 0 (delta 0)
		To git@github.com:yourusername/learngit.git
		   291bea8..adca45d  dev -> dev
4. 多人协作的工作模式通常是这样：
1.首先，可以试图用`git push origin branch-name`推送自己的修改；
2.如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3.如果合并有冲突，则解决冲突，并在本地提交；
4.没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！
如果`git pull`提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream branch-name origin/branch-name`。
这就是多人协作的工作模式，一旦熟悉了，就非常简单。

---
## 8.学习总结
1. 命令行:
`git checkout -b dev` 创建并切换到名为dev的分支
`git branch dev` 创建名为dev的分支
`git checkout dev` 切换到名为dev的分支
`git branch` 查看本地分支列表，当前分支会带一个*号
`git merge dev` 合并指定分支dev到当前分支（非dev分支）
`git branch -d dev`删除指定分支dev(合并后的)
`git branch -D dev`强制删除指定分支dev（合没合并都删）
`git branch -r` 查看远程分支
`git branch -a` 查看所有分支
`git log --graph`查看分支合并图：
`git log --graph --pretty=oneline --abbrev-commit`查看分支（commit）历史（带合并图）
`git merge --no-ff -m "..." dev` 不使用Fast forward的合并，附带一次提交
`git stash` 保存当前工作现场
`git stash list` 查看保存列表
`git stash apply stash@{n}` 恢复保存列表的第n+1项保存现场
`git stash drop stash@{n}` 删除保存列表的第n+1项保存现场
`git stash pop` 恢复并删除最近的添加到栈中的储存现场
`git remote` 查看远程库信息
`git remote add origin-name git@github.com:yourusername/projectAddress.git`添加远程库
`git remote rename oldname newname` 远程仓库重命名
`git remote rm origin-name` 删除远程库
`git push origin dev` 当前分支代码推送到origin远程库的dev分支
（若要在远程创建dev分支，则本地切换到dev分支后使用这句）
`git pull origin` 抓取远程库中和当前分支相关联的分支的最新代码
`git branch --set-upstream dev origin/dev`创建本地dev与远程库dev的链接

2. 学习地址:
<https://www.liaoxuefeng.com/>

---
