---
title: "Gin使用https"
date: 2020-09-13T09:11:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- gin
- https
---

```go
package main

import (
   "fmt"
   "github.com/gin-gonic/gin"
)
import "github.com/unrolled/secure"

func loadTsl() gin.HandlerFunc {
   return func(context *gin.Context) {
      sec := secure.New(secure.Options{SSLRedirect: true, SSLHost: "localhost:8080"})
      err := sec.Process(context.Writer, context.Request)
      if err != nil {
         fmt.Println(err)
         return
      }
      context.Next()
   }
}

func main() {
   engine := gin.Default()
   engine.Use(loadTsl())
   engine.GET("/", func(context *gin.Context) {
      context.JSON(200, gin.H{
         "msg": "success",
      })
   })
   engine.RunTLS(":8080", "helloWorld/certClient/client.pem", "helloWorld/certClient/client.key")
}
```

