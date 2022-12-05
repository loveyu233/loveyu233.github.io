---
title: "Go使用令牌桶"
date: 2020-01-14T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- tokenBucket
---

>令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。

# 创建使用

```go
package tokenBucket

import (
   "log"
   "sync"
   "time"
)

type TokensBucket struct {
   limiter float64    //速率
   burst   int        //桶大小
   mu      sync.Mutex //锁
   tokens  float64    //桶里面的令牌数量
   last    time.Time  //最后一次消耗令牌的时间
}

// NewTokensBucket 创建令牌桶
func NewTokensBucket(limiter float64, burst int) *TokensBucket {
   return &TokensBucket{limiter: limiter, burst: burst}
}

// Allow 使用，每次消耗一个令牌
func (t *TokensBucket) Allow() bool {
   return t.AllowN(time.Now(), 1)
}

// AllowN 当前时间，一次消耗的令牌
func (t *TokensBucket) AllowN(now time.Time, i int) bool {
   t.mu.Lock()
   defer t.mu.Unlock()
   //当前时间-最后一次添加令牌的时间 * 桶速率 = 应该补充的令牌
   delta := now.Sub(t.last).Seconds() * t.limiter
   t.tokens += delta
   //桶内令牌 > 桶总大小  =  只补充最大令牌数
   if t.tokens > float64(t.burst) {
      t.tokens = float64(t.burst)
   }
   //桶内令牌 < 需要的令牌 = 返回false
   if t.tokens < float64(i) {
      return false
   }
   //否则返回true，并用桶的剩余令牌 - 消耗令牌
   t.tokens -= float64(i)
   //桶最后一次补充时间重置为当前时间
   t.last = now
   //返回true
   return true
}

```

# 测试

```go
func main() {
   bucket := NewTokensBucket(3, 5)
   for true {
      n := 4
      for i := 0; i < n; i++ {
         go func(i int) {
            if bucket.Allow() {
               log.Printf("allow [%d]", i)
            } else {
               log.Printf("forbid [%d]", i)
            }
         }(i)
      }
      time.Sleep(time.Second)
      log.Println("========================================")
   }
}

```

# gin基本使用

```go
func main() {
	app := gin.Default()
	bucket := tokenBucket.NewTokensBucket(1, 2)
	app.Use(func(context *gin.Context) {
    
   //拿到令牌就给放行
		if bucket.Allow() {
			context.Next()
    //拿不到就不给过
		} else {
			context.JSON(500, gin.H{
				"msg": "false",
			})
			context.Abort()
		}
	})
	app.GET("/", func(context *gin.Context) {
		context.JSON(200, gin.H{
			"msg": "success",
		})
	})
	app.Run(":80")
}

```

