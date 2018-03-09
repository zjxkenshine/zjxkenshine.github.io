---
title: Git命令行总结及常见问题的解决，不断更新
date: 2018-03-08 19:50:51
tags: Git
categories: 版本控制工具

---
## 1.命令行
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
`$ ssh-keygen -t rsa -C "youremail@example.com"`设置ssh密钥
`git remote add origin address` 添加远程仓库
`git push -u origin master` 将代码推送到远程库的master并与之关联
`git push origin master`将当前分支推送到远程库的master。
`git clone address` 从address处克隆一个库到本地
`git log --pretty=oneline --abbrev-commit`查看历史提交，无分支
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
`git remote add address`添加远程库
`git remote rename oldname newname` 远程仓库重命名
`git remote rm origin-name` 删除远程库
`git push origin dev` 当前分支代码推送到origin远程库的dev分支
（若要在远程创建dev分支，则本地切换到dev分支后使用这句）
`git pull origin` 抓取远程库中和当前分支相关联的分支的最新代码
`git branch --set-upstream dev origin/dev`创建本地dev与远程库dev的链接
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
`$ git config --global color.ui true` 配置git颜色
`git add -f file ` 强制添加一个文件
`git check-ignore -v file` 检查.gitignore文件是否忽略file
`git config --global alias.othername oldname` 配置别名
`git config --global alias.othername 'string'` 配置别名

---
## 2.我遇到的问题列表
1. `git log/show/diff`信息过多怎么返回(退出)?
2. `git commit`忘记加-m报错怎么解决?

---
## 3.问题解决
1. `git log/show/diff`信息过多怎么返回(退出)：
方法1：按ctrl+C;
方法2：英文状态下按q。
2. `git commit`忘记加-m报错怎么解决：
方法1：关掉重启Git Bash
方法2：输入:wq然后回车

---