---
title: "Go Swagger"
date: 2021-07-12T09:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- swagger
---

## 一：main方法注释

```go
// @title 这里写标题
// @version 1.0
// @description 这里写描述信息
// @termsOfService http://swagger.io/terms/

// @contact.name 这里写联系人信息
// @contact.url  这里写联系人信息url
// @contact.email 这里写联系人信息email

// @license.name Apache 2.0开源协议
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html协议地址

// @host 这里写接口服务的host
// @BasePath 这里写base path
func main() {
	r := gin.New()

	// 注册swagger服务
	r.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))
	r.Run("127.0.0.1:9001")
}
```



## 二：请求接口注释

```go
// ShowAccount godoc
// @Summary      接口概要
// @Description  接口描述
// @Tags         标签
// @Accept       application/json
// @Produce      application/json
//@Param 参数格式，从左到右分别为：参数名、入参类型、数据类型、是否必填、注释
//@Success 响应成功，从左到右分别为：状态码、参数类型、数据类型、注释
//@Failure 响应失败，从左到右分别为：状态码、参数类型、数据类型、注释
// @Router       /accounts/show [get]
func ShowAccount(ctx *gin.Context) {

}
```

>### 参数类型
>
>- query
>- path
>- header
>- body
>- formData
>
>### 数据类型
>
>- string (string)
>- integer (int, uint, uint32, uint64)
>- number (float32)
>- boolean (bool)
>- user defined struct



## 三: 生成文档

```shell
swag init -g main.go
```



## 四: 引用

```go
_ "swaggerDemo/docs"
```



## 五: 载入gin

```go
engine.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```



## 六: 访问index页面

地址/swagger/index.html
