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



```go
package jwt

import (
	"github.com/golang-jwt/jwt/v4"
	"github.com/spf13/viper"
	"time"
)

type Claims struct {
	UserId int
	jwt.RegisteredClaims
}

func CreateToken(id int) (string, error) {
	accessJwtKey := []byte(viper.GetString("token.secret"))
	expirationTime := time.Now().Add(60 * time.Minute) // 5分钟有效

	claims := Claims{
		UserId: id,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(expirationTime),
			Issuer:    "233",
		},
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, &claims)
	return token.SignedString(accessJwtKey)
}

func GetPayload(token string) (int, error) {
	parser := jwt.NewParser()
	var claims Claims
	_, _, err := parser.ParseUnverified(token, &claims)
	return claims.UserId, err
}

func VerifyToken(token string) error {
	accessJwtKey := []byte(viper.GetString("token.secret"))
	_, err := jwt.Parse(token, func(token *jwt.Token) (interface{}, error) {
		return accessJwtKey, nil
	})
	return err
}

```

