---
title: "Zap"
date: 2020-01-04T11:31:31+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- zap
---

# zap

## 日志配置

```go
func getZapConfig() zapcore.EncoderConfig {
	c := zap.NewProductionEncoderConfig()
	c.EncodeTime = func(t time.Time, pae zapcore.PrimitiveArrayEncoder) {
		pae.AppendString(t.Format(("[GIN_ZVCJ] " + "2006/01/02 - 15:04:05.000")))
	}
	c.EncodeLevel = func(l zapcore.Level, pae zapcore.PrimitiveArrayEncoder) {
		pae.AppendString("[" + l.CapitalString() + "]")
	}
	c.EncodeCaller = func(ec zapcore.EntryCaller, pae zapcore.PrimitiveArrayEncoder) {
		pae.AppendString("[" + ec.TrimmedPath() + "]")
	}
	return c
}
```



## 日志分割

```go
func getZapWrite(filename string) zapcore.WriteSyncer {
	lumberjack := &lumberjack.Logger{
		Filename:   filename,
		MaxSize:    10,
		MaxAge:     30,
		MaxBackups: 5,
		Compress:   true,
	}
	return zapcore.AddSync(lumberjack)
}
```



## 创建core

```go
func newZapCore(filename string, level zapcore.LevelEnabler) zapcore.Core {
	return zapcore.NewCore(getZapEncode(), getZapWrite(filename), level)
}

func getZapEncode() zapcore.Encoder {
	return zapcore.NewConsoleEncoder(getZapConfig())
}
```



## 日志初始化

```go
func ZapInit() *zap.Logger {
	info := zap.LevelEnablerFunc(func(l zapcore.Level) bool {
		return l == zap.InfoLevel
	})
	debug := zap.LevelEnablerFunc(func(l zapcore.Level) bool {
		return l == zap.DebugLevel
	})
	warn := zap.LevelEnablerFunc(func(l zapcore.Level) bool {
		return l == zap.WarnLevel
	})
	error := zap.LevelEnablerFunc(func(l zapcore.Level) bool {
		return l >= zap.ErrorLevel
	})

	t := time.Now().Format("2006-01-02")
	cores := [...]zapcore.Core{
		newZapCore(fmt.Sprintf("%s/%s/info.log", "log", t), info),
		newZapCore(fmt.Sprintf("%s/%s/debug.log", "debug", t), debug),
		newZapCore(fmt.Sprintf("%s/%s/warn.log", "warn", t), warn),
		newZapCore(fmt.Sprintf("%s/%s/error.log", "error", t), error),
	}
	return zap.New(zapcore.NewTee(cores[:]...), zap.AddCaller())
}
```





# gin使用zap日志



## 配置gin处理

```go
// 日志处理
func GinLogger() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()
		cost := time.Since(start)
		global.GZV_LOG.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// 错误处理
func GinRecovery(stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					global.GZV_LOG.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					c.Error(err.(error))
					c.Abort()
					return
				}

				if stack {
					global.GZV_LOG.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					global.GZV_LOG.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}
```



## 使用

```go
e := gin.Default()
e.Use(config.GinRecovery(true), config.GinLogger())
```
