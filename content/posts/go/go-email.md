---
title: "Go Email"
date: 2020-02-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- email
---

```go
package test

import (
   "crypto/tls"
   "fmt"
   "gopkg.in/gomail.v2"
)

var (
   host     = "smtp.qq.com"
   port     = 25
   password = "邮箱密码"
   username = "邮箱id"
   to       = "发送者的邮箱id"
)

func Send() {
   message := gomail.NewMessage()
   message.SetHeader("From", username)
   message.SetHeader("To", to)
   message.SetHeader("Subject", "TEST")
   message.SetBody("text/html", "<h1 style='color:red'>123</h1>")
   dialer := gomail.NewDialer(host, port, username, password)
   dialer.TLSConfig = &tls.Config{InsecureSkipVerify: true}
   err := dialer.DialAndSend(message)
   if err != nil {
      fmt.Println(err)
   } else {
      fmt.Println("ok")
   }
}
```
