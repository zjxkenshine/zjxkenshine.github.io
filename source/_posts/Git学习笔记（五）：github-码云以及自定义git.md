---
title: Git学习笔记（五）：github,码云以及自定义git
date: 2018-03-08 18:05:51
tags: Git
categories: 版本控制工具

---
## 0.学习要点
- 码云的使用
- git忽略特殊文件
- git配置别名

---
## 1.GitHub
1. 选择一个别人的项目，点“Fork”就在自己的账号下克隆了一个bootstrap仓库，然后使用clone克隆到本地。一定要从自己的账号下clone仓库，这样你才能推送修改。如果你想修复一个bug，或者新增一个功能，立刻就可以开始干活，干完后，往自己的仓库推送。如果你希望官方库能接受你的修改，你就可以在GitHub上发起一个pull request。当然，对方是否接受你的pull request就不一定了。
2. 小结
- 在GitHub上，可以任意Fork开源仓库；
- 自己拥有Fork后的仓库的读写权限；
- 可以推送pull request给官方仓库来贡献代码

---
## 2.码云:
1. 相比github的优点
使用GitHub时，国内的用户经常遇到的问题是访问速度太慢，有时候还会出现无法连接的情况。
如果我们希望体验Git飞一般的速度，可以使用国内的Git托管服务——码云（gitee.com）。
和GitHub相比，码云也提供免费的Git仓库。此外，还集成了代码质量检测、项目演示等功能。对于团队协作开发，码云还提供了项目管理、代码托管、文档管理的服务，5人以下小团队免费。码云的免费版本也提供私有库功能，只是有5人的成员上限。
2. 配置ssh
和github类似，注册登录后，选择右上角用户头像 -> 菜单“修改资料”，然后选择“SSH公钥”，填写一个便于识别的标题，然后把用户主目录下的.ssh/id_rsa.pub文件的内容粘贴进去。
3. 上传项目：
和github操作一毛一样，新建一个仓库，`git remote add origin address`添加远程仓库，`git push origin master`推送代码。只要这个`origin`和github的不同，本地仓库的代码就可以同时提交到github与码云。
若重复，可以使用`git remote rm origin`删除远程库。
![马云](http://p5ki4lhmo.bkt.clouddn.com/00005git%E5%AD%A6%E4%B9%A05-1.jpg)

---
## 3.配置git颜色与忽略特殊文件
1. 配置git颜色：
`$ git config --global color.ui true` 
2. 忽略特殊文件
有些时候，你必须把某些文件放到Git工作目录中，但又不能提交它们，比如保存了数据库密码的配置文件啦，等等，每次git status都会显示Untracked fill..。
解决方法:**在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。**
GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。详情点击[这里](https://github.com/github/gitignore)。
**忽略的原则**是：
1.忽略操作系统自动生成的文件，比如缩略图等；
2.忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；
3.忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。
检验.gitignore的标准是`git status`命令是不是说`working directory clean`。
`.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理！
3. 涉及到忽略文件的命令行
		$ git add -f file
强制添加一个文件（即使它被ignore）
		$ git check-ignore -v file
检查是.gitignore文件否出错

---
## 4.配置别名
1. 基本配置：
用`git st`代替`git status`：
	$ git config --global alias.st status
然还有别的命令可以简写，很多人都用co表示checkout，ci表示commit，br表示branch：
	$ git config --global alias.co checkout
	$ git config --global alias.ci commit
	$ git config --global alias.br branch
以后提交就可以简写成：
		$ git ci -m "bala bala bala...
`--global`参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用。
也可以使用一个单词代替几个单词，如撤销暂存区修改:
		$ git config --global alias.unstage 'reset HEAD'
配置别名后可以写成:
		$ git unstage test.py
效果同`git reset HEAD file`。
2. 其他配置
配置一个git last，让其显示最后一次提交信息：
		$ git config --global alias.last 'log -1'
配置一个很厉害的日志输出:
		$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

---
## 5.配置git服务器
需要linux知识，以后再说 

---
## 6.学习总结
1. 命令行：
`$ git config --global color.ui true` 配置git颜色
`git add -f file ` 强制添加一个文件
`git check-ignore -v file` 检查.gitignore文件是否忽略file
`git config --global alias.othername oldname` 配置别名
`git config --global alias.othername 'string'` 配置别名

2. 学习地址： 
<https://www.liaoxuefeng.com>
---