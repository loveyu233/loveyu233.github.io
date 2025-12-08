---
title: Telepresence
date: 2025-12-08T14:01:04+08:00
author: ["loveyu"]
draft: false
categories: 
- other
tags: 
- telepresence
- k8s
---

# 官网

>https://telepresence.io/docs/install/client



# 使用

## 复制k8s的配置到一个文件

>例如文件名为：k8s_conf

## 链接

>telepresence connect --kubeconfig=k8s_conf

## 退出

>telepresence quit



# 错误处理

>✘ dial to socket /var/run/telepresence-daemon.socket failed: dial unix 9223372036.9s /var/run/telepresence-daemon.socket: context deadline exceeded; remove of unresponsive socket failed: remove /var/run/telepresence-daemon.socket: permission denied; this usually means that the process has locked up

## 停掉 telepresence 相关进程

- telepresence quit || true
- sudo pkill -f telepresence-daemon || true

## 删除残留的 socket

- sudo rm -f /var/run/telepresence-daemon.socket

## 重新连接
