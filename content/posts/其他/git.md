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

