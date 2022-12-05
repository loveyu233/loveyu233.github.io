---
title: "go使用Jwt"
date: 2020-04-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags:
- go
- jwt
---



1. #### 导入包

>"github.com/dgrijalva/jwt-go"

2. #### 声明结构体

>```go
>type MyStandardClaims struct {
>Username string `json:"username"`
>jwt.StandardClaims
>}
>```
>
> 必须添加属性：jwt.StandardClaims 
>
>可以添加自己需要的如：Username

3. #### 设置密钥

>myKey := []byte("qwertyuiop") //密钥尽量长一点，过短会报错密钥过短

4. #### 创建结构体

>```go
>ms := MyStandardClaims{
>Username: "loveyu",
>StandardClaims: jwt.StandardClaims{
> ExpiresAt: time.Now().Unix() + 10,
> Issuer:    "233",
>},
>}
>```
>
> StandardClaims: jwt.StandardClaims{}  可以添加其他属性
>
>ExpiresAt：过期时间
>
>Issuer：生成jwt的所有者

5. #### 创建token

>```go
>token := jwt.NewWithClaims(jwt.SigningMethodHS256, &ms)
>```
>
> jwt.SigningMethodHS256  加密方式

6. #### 加密token

>```go
>signedString, err := token.SignedString(myKey)
>```
>
> signedString  就是需要的jwt字符串
>
>myKey：密钥

7. #### 解析token

>```go
>claims, err := jwt.ParseWithClaims(signedString, &MyStandardClaims{}, func(token *jwt.Token) (interface{}, error) {
>return myKey, nil
>})
>if err != nil {
>fmt.Println(err)
>}
>fmt.Println(claims.Claims.(*MyStandardClaims).Username)
>```
>
> singnedString  为jwt字符串
>
> &MyStandardClaims{}  为解析后的结构体
>
> func(token *jwt.Token)  该函数的返回值为密钥
>
> claims.Claims.(*MyStandardClaims).Username  断言为对应结构体，并获取对应属性的值
