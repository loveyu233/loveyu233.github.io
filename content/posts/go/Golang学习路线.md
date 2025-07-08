---
title: "Go学习路线"
date: 2023-08-27T17:44:56+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
---

<h1><center>Golang学习路线</center></h1>

# 基础

## 基础语法

[刘丹冰老师教学视频：8小时golang转职之路](https://www.bilibili.com/video/BV1gf4y1r79E/?vd_source=c82892ceed4b858d308c45829c725d98)



## Mysql

### 索引事务

[周瑜老师 b 站解说视频：mysql 技术原理解析](https://www.bilibili.com/video/BV1kA411u734/?vd_source=c82892ceed4b858d308c45829c725d98)

## Gorm入门

[李文周老师技术博客：gorm 框架入门篇](https://www.liwenzhou.com/posts/Go/gorm/)

[李文周老师技术博客：gorm 框架操作教程](https://www.liwenzhou.com/posts/Go/gorm-crud/)

[推荐书籍：MySQL技术内幕-InnoDB存储引擎(第2版) 姜承尧 著](https://book.douban.com/subject/24708143/)



## Redis

[官方指令文档](https://redis.io/commands/)

[指令文档中文版](http://www.redis.cn/commands.html)

[redigo：golang redis 开源SDK](https://github.com/gomodule/redigo)

[技术博客：golang 分布式锁 1-3章](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484313&idx=1&sn=905342d3fc1cdaf845a62a140bd5247e)

[b 站解说视频：golang 分布式锁 p1-p4](https://www.bilibili.com/video/BV1Pm4y1b76u/?vd_source=c82892ceed4b858d308c45829c725d98)



## MQ

[技术博客：基于 redis 实现 mq](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484622&idx=1&sn=e5199a4d0276a9aef8630ac9f781c48f)

> kafka/rocketmq/rabbitmq至少了解其中之一的架构及原理

[nsq 开源地址,【进阶】基于 golang 实现的 mq 组件 nsq](https://github.com/nsqio/nsq)



## WEB

### Gin

[李文周老师技术博客：gin 介绍及使用](https://www.liwenzhou.com/posts/Go/gin/)

### 标准库

[技术博客：net/http 底层实现原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484040&idx=1&sn=b710f4429188ea5f49f6a9155381b67f)

[b 站解说视频：net/http 底层实现原理](https://www.bilibili.com/video/BV1Hj411U7Zw/?vd_source=c82892ceed4b858d308c45829c725d98)

### io多路复用,epoll

[技术博客：golang io 与 epoll](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484057&idx=1&sn=50e57108f736bc47137ac57dfb643893)

[b 站解说视频：golang io 与 epoll](https://www.bilibili.com/video/BV1ph4y1f7gr/?vd_source=c82892ceed4b858d308c45829c725d98)

## Gin实现原理

[技术博客：gin 框架实现原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484076&idx=1&sn=9492d326c820625700345a881b58a849)

[b 站解说视频：gin 框架实现原理](https://www.bilibili.com/video/BV1zm4y177mb/)

## Context

[技术博客：context 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483677&idx=1&sn=d1c0e52b1fd31932867ec9b1d00f4ec2)

[b 站解说视频：context 底层原理](https://www.bilibili.com/video/BV1EA41127Q3/?vd_source=c82892ceed4b858d308c45829c725d98)



# 基础知识进阶

## GMP原理

[技术博客：gmp 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483889&idx=1&sn=dd5066f7c27a6b29f57ff9fecb699d77)

[b 站解说视频：gmp 底层原理](https://www.bilibili.com/video/BV1oT411Y7m3/)



## GC

### 内存管理

[技术博客：内存分配与管理机制](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483971&idx=1&sn=409fbc90cd37cd9856f470a0db884218)

### GC原理

[技术博客：gc 原理剖析](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484000&idx=1&sn=e5050d2a63068edef20f0198674e672a)

### GC源码

[技术博客：gc 源码解析](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484011&idx=1&sn=494c5f1aff5ecac8a9eee26bf7c00c85)



## Channel

[技术博客：channel 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483770&idx=1&sn=fa999e22d5de4624544488562d6f799d)

[b 站解说视频：channel 底层原理](https://www.bilibili.com/video/BV1uv4y187p6/)



## 互斥锁/读写锁

[技术博客：单机锁底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483797&idx=1&sn=34274d44bced0835ea302376a137219b)

[b 站解说视频：单机锁底层原理](https://www.bilibili.com/video/BV1kv4y157wj/)



## Map

[技术博客：map 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483868&idx=1&sn=6e954af8e5e98ec0a9d9fc5c8ceb9072)

[b 站解说视频：map 底层原理](https://www.bilibili.com/video/BV1Es4y1y7UD/)



## sync.Map

[技术博客：sync.Map 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483821&idx=1&sn=f45e9e2b4c4cb7edaa57d904e3bf7bd7)

[b 站解说视频：sync.Map 底层原理](https://www.bilibili.com/video/BV1uk4y1v7Cb/?vd_source=c82892ceed4b858d308c45829c725d98)



## Slice

[技术博客：slice 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484378&idx=1&sn=3d2d4a0055a96c8c76e620371bfec7f7)

[b 站解说视频：slice 底层原理](https://www.bilibili.com/video/BV1Bz4y1s7g1/?vd_source=c82892ceed4b858d308c45829c725d98)



## sync.WaitGroup

[技术博客：waitGroup 底层原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484367&idx=1&sn=6894cb1cf2d5741fe5ce15e989e7a017)



# 分布式理论

## 分布式锁

### golang 分布式锁

[技术博客：go 分布式锁](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484313&idx=1&sn=905342d3fc1cdaf845a62a140bd5247e)

[b 站解说视频：go 分布式锁](https://www.bilibili.com/video/BV1Pm4y1b76u/?vd_source=c82892ceed4b858d308c45829c725d98)

### redis 分布式锁进阶

[技术博客：redis 分布式锁进阶](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484339&idx=1&sn=d52b8e1182184394f49815c631c0e074)

[b 站解说视频：redis 分布式锁进阶](https://www.bilibili.com/video/BV1wP411X7sc/?vd_source=c82892ceed4b858d308c45829c725d98)

## 共识算法

### raft

[技术博客：raft 原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483922&idx=1&sn=8fffd743acc20b4a682346c901924dd6)

[b 站解说视频：raft 原理](https://www.bilibili.com/video/BV1Kz4y1H7gw/?vd_source=c82892ceed4b858d308c45829c725d98)

### raft-etcd

[技术博客：raft-etcd 源码](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483950&idx=1&sn=cdd4a6d5dca608a6b4ea91a7f943bd41)

[b 站解说视频：raft-etcd 源码](https://www.bilibili.com/video/BV1wV4y117NE/?vd_source=c82892ceed4b858d308c45829c725d98)

## 分布式事务

### 原理

[技术博客：分布式事务原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484585&idx=1&sn=b5ee56c2334e3cf4e9a1d8d9b54cd02c)

[b 站解说视频：分布式事务原理](https://www.bilibili.com/video/BV1R14y1q7WX/?vd_source=c82892ceed4b858d308c45829c725d98)

### 手写tcc框架

[技术博客：手写 tcc](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484619&idx=1&sn=2415f0b9c1e043c22ae2fd6d75d6cbb3)

[b 站解说视频：手写 tcc](https://www.bilibili.com/video/BV1ku4y1Q7GN/?vd_source=c82892ceed4b858d308c45829c725d98)

## 微服务框架

[go-zero 学习攻略](https://github.com/Ouyangan/go-zero-annotation)



# 优秀的开源项目

## 协程池ants

[github 开源地址](https://github.com/panjf2000/ants)

[技术博客：解析 ants 原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483771&idx=1&sn=dba806a7c991f75a60e1fe69876e6c3f)

[b 站解说视频：解析 ants 原理](https://www.bilibili.com/video/BV1wk4y1h7k7/?vd_source=c82892ceed4b858d308c45829c725d98)

## grpc-go

[github 开源地址](https://github.com/grpc/grpc-go)

[技术博客：grpc-go 服务端原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484144&idx=1&sn=5e60c1e17d7c905c3e92fa9774e0331c)

[技术博客：grpc-go 客户端原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484161&idx=1&sn=ec088c67d556acb922f291f1b74e5282)



## ETCD

### 服务注册与发现

[技术博客：基于 etcd 实现 grpc-go 服务注册与发现](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484251&idx=1&sn=06e4452cecdcea48de27e26ab812676a)

### watch监听机制

[技术博客：etcd watch 客户端篇](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484269&idx=1&sn=a23e9a323f23d801395559feed844b6f)

[技术博客：etcd watch 服务端篇](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484285&idx=1&sn=230b3574363d861e7af3ce0d34ae8733)

### etcd-raft

[技术博客：raft-etcd 工程案例](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247483950&idx=1&sn=cdd4a6d5dca608a6b4ea91a7f943bd41)

[b 站解说视频：raft-etcd 工程案例](https://www.bilibili.com/video/BV1wV4y117NE/?vd_source=c82892ceed4b858d308c45829c725d98)



# 数据结构与算法

## skiplist跳表

### 普通跳表

[技术博客：从零到一写跳表](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484204&idx=1&sn=54817591aa44359cde9b1b88d386b31b)

[b 站解说视频：从零到一写跳表](https://www.bilibili.com/video/BV1fP411B71T/?vd_source=c82892ceed4b858d308c45829c725d98)

### 并发跳表

[技术博客：实现并发跳表](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484228&idx=1&sn=f30aa1ec0ae19d934beaa3ab13e2c3ee)

## lsm tree日志结构合并树

[技术博客：初探 rocksDB](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484182&idx=1&sn=6ec38965bc927bf72eee567342f6376a)

## bloom filter布隆过滤器

[技术博客：布隆过滤器原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484535&idx=1&sn=bf906823a8ac5efd1a7c567622ff8968)

[b 站解说视频：布隆过滤器原理](https://www.bilibili.com/video/BV1B44y1c7E1/?vd_source=c82892ceed4b858d308c45829c725d98)

## trie前缀树

[技术博客：实现前缀树 trie](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484473&idx=1&sn=3c4e3f0b5b4ca29ebc4f317f825f73ed)

## geohash

[技术博客：geohash 原理](https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484507&idx=1&sn=ba2c821a75660b7aa6ec9e150b93f06e)

## 一致性hash