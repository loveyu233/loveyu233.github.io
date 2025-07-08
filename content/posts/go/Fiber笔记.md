---
title: "fiber基础"
date: 2023-07-10T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- fiber
---

# 获取参数

Params: 获取url参数

Query: 获取get请求参数

Get: 获取请求头参数

FormValue: 获取表单参数

BodyParser: 获取json格式参数

FormFile: 获取文件类型参数, 配合SaveFile使用

<br/>

<br/>

# APP

Name: 给路由起一个名字

GetRoute: 获取指定name的路由信息

Stack: 获取全部的路由信息

<br/>

Mount: 挂载

MountPath: 获取挂载后的路由path地址

<br/>

GetRoutes: 参数为true过滤中间件,false则不过滤

<br/>

ListenTLS: 用于设置https, 参数分别为地址,pem,key

<br/>

hooks := app.Hooks():

onRoute 是请求路由注册时的钩子函数切片。可以通过 app.OnRoute() 方法向切片中添加钩子函数，这些函数将在每次调用 app.Method() 注册路由时触发。

onName 是命名路由注册时的钩子函数切片。可以通过 app.OnName() 方法向切片中添加钩子函数，这些函数将在每次调用 app.Get("/path", handler).Name("name") 设置命名路由时触发。

onGroup 是路由组注册时的钩子函数切片。可以通过 app.OnGroup() 方法向切片中添加钩子函数，这些函数将在每次调用 app.Group("/prefix") 创建路由组时触发。

onGroupName 是命名路由组注册时的钩子函数切片。可以通过 app.OnGroupName() 方法向切片中添加钩子函数，这些函数将在每次调用 app.Group("/prefix").Name("name") 设置命名路由组时触发。

onListen 是应用程序监听端口时的钩子函数切片。可以通过 app.OnListen() 方法向切片中添加钩子函数，这些函数将在调用 app.Listen(address) 时触发。

onShutdown 是应用程序关闭时的钩子函数切片。可以通过 app.OnShutdown() 方法向切片中添加钩子函数，这些函数将在调用 app.Shutdown() 时触发。

onFork 是应用程序分叉处理时的钩子函数切片。可以通过 app.OnFork() 方法向切片中添加钩子函数，这些函数将在调用 app.Fork() 时触发。

onMount 是应用程序挂载子应用程序时的钩子函数切片。可以通过 app.OnMount() 方法向切片中添加钩子函数，这些函数将在调用 app.Mount("/prefix", subApp) 时触发。

<br/>

# CTX

Params: 获取路径参数

AllParams: 获取全部

<br/>

Append: 添加请求响应头信息

<br/>

Download: 设置下载的文件和指定文件的名称,可以指定中文无需其他设置

<br/>

Locals: 上下文中添加kv, 只有一个参数则是获取参数对应的value值

<br/>

Redirect: 重定向

<br/>

# middleware

BasicAuth,Cache,CORS,CSRF

<br/>

<br/>

# Other

Content-Disposition: 响应头字段,用于指定浏览器是展示资源还是下载资源

内容显示：通过设置 Content-Disposition: inline，浏览器可以将响应的内容直接显示在页面上，例如显示图片、视频、PDF等文件。

下载保存：通过设置 Content-Disposition: attachment，浏览器会将响应的内容作为下载文件，而不是直接在浏览器中打开。这对于提供下载链接非常有用，使用户可以保存文件到本地或者查看下载进度。
