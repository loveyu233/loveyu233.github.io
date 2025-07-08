---
title: "go项目上传调用"
date: 2024-03-20T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- github
- gitlab
---

## gitlab/github创建仓库

## 创建目录

>创建的目录名称要和gitlab/github创建的仓库名称一致

```shell
mkdir 仓库名称
```



## 初始化项目

>### 如果是gitlab默认用户名有大小写,无论大小写统一全部按照小写 ZhangSan --> zhangsan
>
>### `初始化的项目名必须格式正确`，不能只写一个项目名称，必须添加上远程仓库地址和用户名/群组名

```shell
go mod init gitlab地址或github.com/用户名或群组名/仓库名
# 例如
go mod init 123.123.123.123/zhangsan/common-go
go mod init github.com/zhangsan/common-go
```



## 创建go文件

>### 对外的函数记得``大写`` 

## 上传

>1. ### 标签必须是`v开头且是三组,例如v1.0.0 v1.2.3 v2.2.2`

```shell
git init --initial-branch=main
git remote add origin ssh://git@123.123.123/zhangsan/common-go.git
git add .
git common -m "init"
git push -u origin main
git tag v1.0.0
git push --tags
```



## 调用

>### 第一次调用某个函数尽量用:`gitlab或github地址/用户名或群组名/仓库名/要调用的包名`  例如: `123.123.123/zhangsan/common-go/redis`
>
>### 如果在goland的项目中直接`import "123.123.123/zhangsan/common-go"` 然后鼠标悬浮上边点击同步依赖会`报错 but does not contain package`
>
>### 需要`import "123.123.123/zhangsan/common-go/redis"` 然后再鼠标点击同步依赖就没问题了

## 仓库更新

>### 正常更新代码，然后提交远程仓库，然后打tag（注意格式）
>
>### 如果调用方想要使用最新的则使用`go get 123.123.123/zhangsan/common-go@latest`即可