---
title: "Gin文件下载乱码"
date: 2020-06-14T14:18:17+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gin
---

>设置Content-Type和Content-Disposition

```go
engine := gin.Default()
engine.GET("/up", func(c *gin.Context) {
	c.Header("Content-Type", "application/octet-stream")
	c.Header("Content-Disposition", fmt.Sprintf("attachment; filename*=utf-8''%s", url.QueryEscape("哈哈.png")))
	file, err := ioutil.ReadFile("file/abc.png")
	if err != nil {
		log.Println(err)
	}
	c.Writer.Write(file)
})
engine.Run("127.0.0.1:8888")

```

