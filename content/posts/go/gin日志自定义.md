---
title: "Gin日志自定义"
date: 2020-02-01T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gin
---

>必须设置文件有写权限：0777

```go
var logFile io.Writer

func init() {
   logFile, _ = os.OpenFile("./log/log.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0777)
   log.SetOutput(io.MultiWriter(logFile, os.Stdout))
   log.SetFlags(log.Ldate | log.Lshortfile | log.Lmicroseconds)
}
func main() {
   app := gin.Default()
   // 非错误日志写入的文件
   gin.DefaultWriter = io.MultiWriter(logFile, os.Stdout)
   // 错误日志写入的文件
   gin.DefaultErrorWriter = io.MultiWriter(logFile, os.Stdout)
   // debug输出格式
   gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
      log.Printf("endpoint %v %v %v %v\n", httpMethod, absolutePath, handlerName, nuHandlers)
   }
   // 自定义日志格式
   app.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
      return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
         param.ClientIP,
         param.TimeStamp.Format(time.RFC1123),
         param.Method,
         param.Path,
         param.Request.Proto,
         param.StatusCode,
         param.Latency,
         param.Request.UserAgent(),
         param.ErrorMessage,
      )
   }))

   err := app.Run("127.0.0.1:8888")
   if err != nil {
      panic(err)
   }
}

```

