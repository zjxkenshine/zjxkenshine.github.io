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


---
## 6.操作标签

---
## 7.学习总结

---