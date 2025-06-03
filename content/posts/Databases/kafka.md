---
title: "kafka"
date: 2020-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- kafka
---

# 指令

|         参数         |                   描述                   |
| :------------------: | :--------------------------------------: |
|  --bootstrap-server  |         连接Kafka主机名称和端口          |
|       --alter        |                 修改主题                 |
|       --create       |                 创建主题                 |
|       --delete       |                 删除主题                 |
|      --describe      |               主题详细信息               |
|     --partitions     | 设置分区数，分区数修改只能大于当前分区数 |
| --replication-factor |        设置分数副本，无法直接修改        |
|        --list        |               查看全部主题               |
|       -- topic       |              操作的主题名称              |

## 创建topic

>--bootstrap-server：指定链接的Kafka，多个用逗号隔开
>
>--topic：指定topic名称
>
>--create：指定为创建topic
>
>--partitions：分区数
>
>--replication-factor：备份数，不能大于当前集群数，单节点就是1

```shell
~ > kafka-topics --bootstrap-server 127.0.0.1:9092 --topic first --create --partitions 1 --replication-factor 1
Created topic first.
```



## 查看topic

>--list：查看有哪些topic
>
>--describe：查看某一个topic的信息

```shell
~ > kafka-topics --bootstrap-server 127.0.0.1:9092 --list
first

~ > kafka-topics --bootstrap-server 127.0.0.1:9092 --topic first --describe
Topic: first	TopicId: XMyZmRawTo6_DGxtYS7v1A	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: first	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```



## 修改分区数

>修改的分区数只能大于原来的分区数

```shell
~ > kafka-topics --bootstrap-server 127.0.0.1:9092 --topic first --alter --partitions 3

==修改后

~ > kafka-topics --bootstrap-server 127.0.0.1:9092 --topic first --describe
Topic: first	TopicId: XMyZmRawTo6_DGxtYS7v1A	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: first	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: first	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: first	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

## 成产

```shell
~ > kafka-console-producer --bootstrap-server localhost:9092 --topic first
```



## 消费

```shell
== 从执行开始接受发送的数据，未开始执行前生产者发送的数据不接受
~ > kafka-console-consumer --bootstrap-server localhost:9092 --topic first

== 接受全部数据，包括未执行前发送的数据
~ > kafka-console-consumer --bootstrap-server localhost:9092 --topic first --from-beginnin
g
```

