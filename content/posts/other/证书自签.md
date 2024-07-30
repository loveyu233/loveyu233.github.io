---
title: "ssl/tls证书自签"
date: 2020-05-21T09:18:25+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- 安全证书
---

> go1.15以后必须是SAN证书

>需要下载openssl
>
>mac下载方法：brew install openssl

### 生成 CA 根证书

> genrsa -out ca.key 2048



> req -new -x509 -days 3650 -key ca.key -out ca.pem



### 生成服务端证书

> genpkey -algorithm RSA -out server.key



> req -new -nodes -key server.key -out server.csr -days 3650 -subj "/C=cn/OU=custer/O=custer/CN=localhost" -config ./openssl.cnf -extensions v3_req



> x509 -req -days 3650 -in server.csr -out server.pem -CA ca.pem -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req



### 生成客户端证书

> genpkey -algorithm RSA -out client.key



> req -new -nodes -key client.key -out client.csr -days 3650 -subj "/C=cn/OU=custer/O=custer/CN=localhost" -config ./openssl.cnf -extensions v3_req



> x509 -req -days 3650 -in client.csr -out client.pem -CA ca.pem -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req
