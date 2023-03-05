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

