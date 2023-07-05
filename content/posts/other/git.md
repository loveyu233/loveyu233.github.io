---
title: "git"
date: 2020-08-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- git
---



```shell
git init：在当前目录初始化一个新的 Git 仓库。

git clone <repository>：克隆远程代码仓库到本地。

git add <file(s)>：将文件添加到暂存区，准备提交。

git commit -m "<message>"：提交暂存区的改动，并添加一条提交信息。

git status：显示工作区和暂存区的状态。

git diff：显示当前未暂存的改动。

git diff --staged：显示已暂存但未提交的改动。

git log：显示提交记录。

git branch：显示分支列表。

git branch <branch-name>：创建一个新分支。

git checkout <branch-name>：切换到指定分支。

git checkout -b <branch-name>：创建并切换到一个新分支。

git merge <branch-name>：将指定分支合并到当前分支。

git remote add <remote-name> <repository-url>：添加一个远程代码仓库。

git push <remote-name> <branch-name>：将本地分支推送到远程仓库。

git pull <remote-name> <branch-name>：从远程仓库拉取最新的改动。

git fetch <remote-name>：从远程仓库获取最新的改动，但不合并到当前分支。

git reset <commit>：撤销提交，并将代码回滚到指定的提交。

git revert <commit>：撤销指定的提交，并创建一个新的提交来记录撤销操作。
```





>`git reset` 和 `git revert` 是 Git 中用于撤销提交的两个命令，它们有以下区别：
>
>1. 行为不同：
>   - `git reset`：将分支指针移动到指定的提交，并丢弃指定提交之后的所有提交。这意味着被重置的提交以及其后的提交都将被永久删除，历史记录将被修改。
>   - `git revert`：创建一个新的提交，来撤销指定的提交。这意味着被撤销的提交将被保留在历史记录中，但是通过撤销提交引入的更改将被还原。
>2. 操作范围不同：
>   - `git reset`：可以通过不同的参数控制重置的行为。例如，使用 `--soft` 参数只会将分支指针移动到指定的提交，保留工作区和暂存区的更改。`--mixed` 参数（默认）将移动分支指针并重置暂存区，但保留工作区的更改。`--hard` 参数将分支指针、暂存区和工作区都移动到指定提交。
>   - `git revert`：撤销指定的提交，并将更改应用到当前分支上，生成新的提交。撤销的提交会被还原，但是通过撤销提交引入的更改将通过新的提交进行逆转。
>3. 影响历史记录：
>   - `git reset`：由于重置操作会修改历史记录，因此在共享的仓库中使用时需要小心。如果已经将修改推送到远程仓库，其他人在拉取和合并代码时可能会遇到问题。
>   - `git revert`：撤销提交通过创建新的提交来实现，可以安全地应用于共享的仓库。其他人可以通过拉取和合并新的提交来同步撤销操作。



# 初始化本地仓库

## 有本地仓库

```shell
git remote add origin 仓库地址
git branch -M main
git push -u origin main
```

## 没有本地仓库

```shell
git init
git add .
git commit -m "init project!"
git branch -M main
git remote add origin 仓库地址
git push -u origin main
```

# http链接换git链接

```she
git remote -v
git remote rm origin
git remote -v
git remote add origin git@xxxx.git
git remote -v
```



# 分支操作

>创建分支name

```sh
git branch name

git branch -b name	创建name分支并切换到改分支

git init -b name 	初始化仓库的时候就创建分支name并切换到改分支
```

>修改分支名称

```shell
git branch -m 旧名称 新名称
```



>创建并进入分支

```shell
git checkout -b 分支名称
```



>查看本地分支和远程分支

```shell
git branch -a
```

>推送本地创建的分支到远程

```shell
git push -u origin dev
== 提交全部分支
git push -u origin --all
```

>切换分支

```shell
git checkout -b name
```

>删除本地分支

```sh
git branch -d 本地分支名A	小写d只能删除已合并过的分支,没有合并过的分支要强行删除要使用大写的D
```

>删除远程分支

```shell
git push origin -d 远程分支名
```

>远程分支删除但是本地分支没有同步解决办法

```shell
git remote prune origin
```

>合并分支,把其他的分支合并到当前分支

```shell
git merge 其他分支名称
git rebase 分支名称
区别为:merge和rebase的显示不同,rebase是一条线显示
```



# 日志

>查看日志

```shell
git log

git log --oneline

git reflog 查看本地的引用日志
```

>筛选日志

```shell
git log --before='2021-01-01'	在这个时间之前的包括这个时间

git log --after='2021-01-01'	在这个时间之后的不包括这个时间

git log --author='王'	筛选成员提交,这个时模糊查询的和王有关的都会查询出来

git log --grep='修改'	筛选所有提交信息包括修改的提交
```

>查看修改内容

```shell
git diff 文件名
```



# 从版本库恢复文件

已删除或修改 未add

```shell
git checkout 删除或修改的文件名称
```

已删除或修改 已add但没有commit

```shell
git checkout commitid 删除或修改的文件名称
commitid从git log中查看
```

已删除或修改 已commit

```shell
git checkout commitid 删除或修改的文件名称
commitid从git log中查看
```



# push冲突解决

## 本地分支缺少内容产生的冲突

```shell
git pull
git push
```



## 修改了相同内容产生的冲突

```shell
git pull
手动修改冲突内容
git push
```



## git pull不能解决时:

```shell
git mergetool
```



# git reset

>把已经add的变成未add状态
>
>用于add错了文件想撤回add的文件

```git
git reset --mixed 
```



>工作区变为commitId的版本内容,之前修改的内容会变为已add状态
>
>用于commit后发现commit的文件有问题想撤销commit

```shell
git reset --soft commitId
```



>工作区变为commitId的版本内容,之前修改的内容会消失
>
>用于commit后发现不如上次commit版本好要回退到上个版本

```shell
git reset --hard commitId
```



# git rebase

>合并多次提交,head~3 合并之前的三次提交

```shell
git rebase -i head~3

// 执行命令后:
 1 pick 663816f Update main.go
 2 pick cad7ef5 add a.txt
 3 pick 1b22f75 commit 3 to commit 1
 除去第一行的pick其他的pick改为s即:
  1 pick 663816f Update main.go
  2 s cad7ef5 add a.txt
  3 s 1b22f75 commit 3 to commit 1
 然后就是设置这次合并提交后的提交显示
```





# 错误

>执行git rebase -i head~3错误,解决办法:
>
>git rebase --edit-todo
>
>git commit --amend
>
>git rebase --continue

```shell
@e032b2e5 rebase-i 1/3 > git status
interactive rebase in progress; onto f345896
Last command done (1 command done):
   pick 663816f Update main.go
Next commands to do (2 remaining commands):
   squash cad7ef5 add a.txt
   squash 1b22f75 commit 3 to commit 1
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'main' on 'f345896'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)
```

