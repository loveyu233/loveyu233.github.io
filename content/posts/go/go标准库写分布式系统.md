---
title: "Go标准库写分布式系统"
date: 2023-01-10T08:51:47+08:00
draft: true
---

# 发起注册请求

## 发送的内容

>ServiceName：服务名称
>
>ServiceURL：服务的地址
>
>RequiredServices：当前服务所需要的其他服务
>
>ServiceUpdateURL：注册中心发生变更向服务发送的请求地址

```go
registration := registry.Registration{
   ServiceName:      "client1",
   ServiceURL:       addr,
   RequiredServices: []string{"logService"},
   ServiceUpdateURL: addr + "/services",
}
```

## 怎么发送

>Start：服务启动注册
>
>参数：上下文，地址，端口，注册信息，注册函数
>
>注册函数：
>
>```go
>func ClientOneRegisterHandler() {
>   http.HandleFunc("/test", func(w http.ResponseWriter, r *http.Request) {
>      all, err := io.ReadAll(r.Body)
>      if err != nil {
>         w.WriteHeader(http.StatusInternalServerError)
>         return
>      }
>      w.Write(all)
>   })
>}
>```

```go
ctx, err := service.Start(context.Background(), host, port, registration, ClientOneRegisterHandler)

// Start函数里面
func Start(ctx context.Context, host, port string, reg registry.Registration,
	registerHandlersFun func()) (context.Context, error) {
	// 1.运行注册函数
	registerHandlersFun()
  // 2.启动http服务
	ctx = startService(ctx, reg.ServiceName, host, port)
  // 3.发送注册信息
	err := registry.RegisterClientService(reg)
	if err != nil {
		return ctx, err
	}
	return ctx, nil
}
```

## 获取其他服务

>GetProvider：获取参数的服务地址，返回值为字符串数组

```go
logURL, err := registry.GetProvider("logService")
```



# 注册中心注册服务

