---
title: Git学习笔记（三）：远程仓库与标签管理
date: 2018-03-07 20:01:47
tags: Git
categories: 版本控制工具

---
## 0.学习要点
- 配置SSH
- 连接到远程仓库
- 从远程克隆到本地仓库
- 标签的创建与操作

---
## 1.配置SSH
1. 第一步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了跳过这一步。若没有，在Git Bash下输入:
		$ ssh-keygen -t rsa -C "youremail@example.com"
把邮件地址换成自己的邮件地址,不用设置密码。
`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。
2. 第二步:注册并登陆GitHub，点击settings，再点击SSH and GPG keys，然后点击New SSH key。填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容。
![](/img/00005SSH.jpg "SSH")
3. 为什么添加SSH:
GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。
GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

---
## 2.远程仓库添加，连接和推送
1. 何时需要添加远程仓库:
你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作。
2. 创建远程仓库:
登陆GitHub，然后，在右上角+号找到“New repository”按钮，创建一个新的仓库(learngit)。
3. 连接远程仓库:
		$ git remote add origin git@github.com:yourusername/learngit.git
origin是git默认的远程仓库叫法，可以改成别的，最好和github仓库名一致。`yourusername/learngit.git`是你github的仓库地址加个.git。
`git@github.com:yourusername/learngit.git`就是github仓库的SSH地址。
4. 推送本地代码到远程仓库:
第一次推送，远程仓库是空的时候使用:
		$ git push -u learngit master
		Counting objects: 19, done.
		Delta compression using up to 4 threads.
		Compressing objects: 100% (16/16), done.
		Writing objects: 100% (19/19), 1.70 KiB | 0 bytes/s, done.
		Total 19 (delta 2), reused 0 (delta 0)
		remote: Resolving deltas: 100% (2/2), done.
		To git@github.com:zjxkenshine/learngit.git
		 * [new branch]      master -> master
		Branch master set up to track remote branch master from learngit.
第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
若报错:可能是本地代码的原版本不是最新的。
以后只要用：
		$ git push learngit master
就可以推送本地learngit仓库的master仓库了

---
## 3.SSH警告
当你第一次使用Git的clone或者push命令连接GitHub时，会得到一个警告：

		The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
		RSA key fingerprint is xx.xx.xx.xx.xx.
		Are you sure you want to continue connecting (yes/no)?
为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。
Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：

		Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
若实在担心有人冒充GitHub服务器，可以对照[GitHub的RSA Key的指纹信息](https://help.github.com/articles/github-s-ssh-key-fingerprints/)是否与SSH连接给出的一致。

---
## 4.从远程库克隆
1. 什么时候从远程库克隆：
假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。
2. 新建远程库:
登陆GitHub，创建一个新的仓库。勾选`Initialize this repository with a README`，这样GitHub会自动为我们创建一个`README.md`文件。
注意:此时本地直接push所以会出错。（git上有README.md文件没下载下来）
		To git@github.com:zjxkenshine/MultiThread.git
		 ! [rejected]        master -> master (non-fast-forward)
		error: failed to push some refs to 'git@github.com:zjxkenshine/MultiThread.git'
		hint: Updates were rejected because the tip of your current branch is behind
		hint: its remote counterpart. Integrate the remote changes (e.g.
		hint: 'git pull ...') before pushing again.
		hint: See the 'Note about fast-forwards' in 'git push --help' for details.
3. 克隆本地库
用命令`git clone address`克隆一个本地库：
		$ git clone git@github.com:zjxkenshine/MultiThread.git
		Cloning into 'MultiThread'...
		remote: Counting objects: 3, done.
		remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
		Receiving objects: 100% (3/3), done.
		Checking connectivity... done.
转到MultiThread并打印列表：
		$ cd MultiThread
		$ ls
		README.md

---
## 5.标签简介及创建标签(以下是分支后的内容)
1. 为什么使用标签:
发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。
Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针。因为commit id一大串不好记，可以使用tag查找commit，tag与commit是绑定在一起的。
2. 创建标签:
切换到需要打标签的分支上:
		$ git branch
		* dev
		  master
		$ git checkout master
		Switched to branch 'master'
使用命令`git tag <name>`就可以打一个新标签：
		$ git tag v1.0
可以用命令`git tag`查看所有标签：
		$ git tag
		v1.0
默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打,怎么办,方法是找到历史提交的commit id，然后打上就可以了：
		$ git log --pretty=oneline --abbrev-commit
		97d5cda 合并bug与工作
		3a86388 git 工作完成
		786ba47 bug修复分支合并
		2a8a8ec bug修复
		a59aa3f 不用ff的合并
		2ebb903 测试--no-ff
		7f23e9a 解决冲突
		d80455d master的修改
		2cbbe5e a分支的修改
		bd5dbdf 分支测试
		5b3b482 delete file2.txt
		3c22783 add new file2
		dbb03c7 第四次修改
		ec9e2b9 第三次修改
		7abc597 第二次修改
		1c899ff 第一次修改
		f3b074a create the first git file
若要在解决冲突这里打上一个v0.9可以这样写:
		$ git tag v0.9 7f23e9a
再用命令`git tag`查看标签:
		$ git tag
		v0.9
		v1.0
标签不是按时间顺序列出，而是按字母排序的。
查看标签信息可以用`git show <tagname>`：
		$ git show v0.9
		commit 7f23e9aab68153e0fc1d6c0424fd7cfd5ca33600
		Merge: d80455d 2cbbe5e
		Author: poppy <1754294529@qq.com>
		Date:   Thu Mar 8 12:31:08 2018 +0800
		
		    解决冲突
		
		diff --cc gitBranchFile.txt
		index 96ba9ff,e4f07dc..ba9b436
		--- a/gitBranchFile.txt
		+++ b/gitBranchFile.txt
		@@@ -1,2 -1,2 +1,6 @@@
		  这是分支文件
		- master主分支的修改
		 -a分支的修改
		++<<<<<<< HEAD
		++master主分支的修改
		++=======
		++master分支的修改
		++>>>>>>> a
还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：
		$ git tag -a v0.7 -m "0.7版本" dbb03c7
		$ git show v0.7
		tag v0.7
		Tagger: poppy <1754294529@qq.com>
		Date:   Thu Mar 8 19:04:16 2018 +0800
		
		0.7版本
		
		commit dbb03c7fa0c83a477d70a9b7cce8b32f1e66fecb
		Author: poppy <1754294529@qq.com>
		Date:   Wed Mar 7 17:31:40 2018 +0800
		
		    第四次修改
`git show`后尾巴太长省略，按ctrl+c可以退出。
还可以通过`-s`用私钥签名一个标签：
		$ git tag -s v0.2 -m "signed version 0.2 released" fec145a
签名采用PGP签名，因此，必须首先安装gpg（GnuPG），如果没有找到gpg，或者没有gpg密钥对，就会报错：
		gpg: signing failed: secret key not available
		error: gpg failed to sign the data
		error: unable to sign the tag
如果报错，请参考GnuPG帮助文档配置Key。

---
## 6.操作标签
1. 删除标签:
使用`git tag -d tagname`：
		$ git tag -d v0.8
		Deleted tag 'v0.8' (was 48c291b)
创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。
2. 推送到远程以及从远程删除:
推送某个标签到远程，使用命令`git push origin <tagname>`：
		$ git push learngit v1.0
		Total 0 (delta 0), reused 0 (delta 0)
		To git@github.com:zjxkenshine/learngit.git
		 * [new tag]         v1.0 -> v1.0
一次性推送所有标签到远程`git push origin --tags`:
		$ git push learngit --tags
		Counting objects: 1, done.
		Writing objects: 100% (1/1), 163 bytes | 0 bytes/s, done.
		Total 1 (delta 0), reused 0 (delta 0)
		To git@github.com:zjxkenshine/learngit.git
		 * [new tag]         v0.7 -> v0.7
		 * [new tag]         v0.9 -> v0.9
从远程删除标签，分两步:
1.先删除本地的标签：
		$ git tag -d v0.9
		Deleted tag 'v0.9' (was 7f23e9a)
2.再使用`git push`按如下方式删除标签
		$ git push learngit :refs/tags/v0.9
		To git@github.com:zjxkenshine/learngit.git
		 - [deleted]         v0.9
登录github查看，发现已经删除了v0.9标签。

---
## 7.学习总结
1. 命令
`$ ssh-keygen -t rsa -C "youremail@example.com"`设置ssh密钥
`git remote add origin address` 添加远程仓库
`git push -u origin master` 将代码推送到远程库的master并与之关联
`git push origin master`将当前分支推送到远程库的master。
`git clone address` 从address处克隆一个库到本地
`git tag tagmane` 在最近一次commit创建本地名为tagname的标签
`git tag tagmane commit-id` 在commitid那次提交创建本地名为tagname的标签
`git log --pretty=oneline --abbrev-commit`查看历史提交，无分支
`git tag` 查看标签列表
`git tag -a tagname -m "..." (commit-id)`（在commit-id版本）创建一个标签名为tagname的标签。
`git tag -s tagname -m "..." (commit-id)`创建一个带gpg签名的标签
`git show tagname` 查看tagname标签的所有信息
`git tag -d tagname` 删除本地tagname标签
`git push origin tagname` 推送tagname标签到远程仓库
`git push origin --tags ` 推送所有标签到远程仓库
`git push origin :refs/tags/tagname ` 删除远程tagname标签（需先删除本地）

2. 学习地址:
<https://www.liaoxuefeng.com/>

---