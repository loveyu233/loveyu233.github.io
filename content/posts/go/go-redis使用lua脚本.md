---
title: "Go Redis使用lua脚本"
date: 2020-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- redis
- lua
---

# lua脚本

```lua
return redis.call("set", KEYS[1], ARGV[1])
```

# go-redis使用

```go
package main

import (
	"context"
	"github.com/go-redis/redis/v8"
	"io/ioutil"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr: "127.0.0.1:6379",
	})
	file, _ := ioutil.ReadFile("lua.lua")
	script := redis.NewScript(string(file))
	// 这里你的脚本上有几个key你就在数组里面写几个对应key
	// 最后一个参数是对应你脚本里面的argv，你脚本里面有几个你就写几个
	println(script.Run(context.Background(), client, []string{"name"}, "asd").String())
}
```

