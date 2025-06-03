---
title: "Go非对称加密使用"
date: 2023-05-21T16:20:18+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- ras
- 加密
---



# 对称加密

>加密和解密使用相同的密钥进行加密

## 优点

>计算量小,加密速度快,加密效率高

## 缺点

>密钥一旦泄露那么加密的信息也就不安全了



# 非对称加密

>需要两个密钥: 公开密钥publicKey,和私有密钥privateKey, 公钥和私钥是一对的,使用公钥加密后的信息之后对应的私钥才可以进行解密

## 优点

>安全性高



## 缺点

>加密速度慢,效率不高

## RSA加密过程

- 随机选择两个不相等的质数p和q。p=61，q=53
- 计算p和q的乘积n。n=3222
- 计算n的欧拉函数φ(n) = (p-1)(q-1)。 φ(n) =3120
- 随机选择一个整数e，使得1< e < φ(n)，且e与φ(n) 互质。e=17
- 计算e对于φ(n)的模反元素d，即求解e*d+ φ(n)*y=1。d=2753, y=-15
- 将n和e封装成公钥，n和d封装成私钥。公钥=(3233，17)，公钥=(3233，2753)

## go代码实现加解密

### 一: 使用ssl生成对应的公钥和私钥pem文件

>pem是一种标准格式，它通常包含页眉和页脚

```shell
openssl genrsa -out data/rsa_private_key.pem 1024
openssl rsa -in data/rsa_private_key.pem -pubout -out data/rsa_public_key.pem
```

``rsa_private_key.pem``

```pem
-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQC5vgoY3FkQmR+BM76mFHx4odRPu5Qlz4cYXP1jxY0s27kb9WMW
eokv3+9cHysh+M3U98NlE1Zr+XUBALAWC1V0uIF6KJJuQ/s4PV1mminoVX5rihlc
1G0aG23o43tWmfpXVkjvs/CmhSFoiIRvoHCKSCyTkaifilmz+3bqQJTQiQIDAQAB
AoGBAK06wiwBhcdnJ+zWF57JSHUxaNOb/EVvUW21fFVK76nAmtmqeGmEiuHtlk1y
fEXIyB8xnDhuWpGFLExtGczVcTI7PnM5RBcZZetQ7aAK8ABfVP2upOXXw/wNZfll
+Dki+bt4/ftDptZHahKbg3QaqRCshNn0J6enEUqZCgQ5WisJAkEA9S/ch1pFQrPO
NTMkPlwowDZCJzrSebjGb677r1wlHBbb/hBzIt89Csa6szfV4/dDAa7abYQI1YLI
QxbWHE74EwJBAMHvDbx2lxGGhWuM8OBOVVxy3sKDDFdLulaSTxoqAMSVj1p+Nowm
hYWk/zbndkTTGJgD2UllTKdIN4cicjKjIHMCQQC6mYtXg68Ufa1hNaPOxerJpkGg
g5btxl9XXi/0HMetYgRZjoFht85IJkiu3r6s+WCIpl9cW9ExVZA95uJatwr7AkEA
tH8Nxc6KI+GT49m1hs7hW7393gOiRM1SjKh3vt5BALZCSfMWSbLAqvY6Ipui08O1
LCbI4SrLARaRt9AzgTWaSQJABpKExipUBUFNJFB3/KzNJtY6HYLO8S+a+BMiR75p
e2aLqEytZuPEF+RhXhxNv0y3YaX1kCqjF4ftmWnUZO2XzA==
-----END RSA PRIVATE KEY-----
```

``rsa_public_key.pem``

```pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5vgoY3FkQmR+BM76mFHx4odRP
u5Qlz4cYXP1jxY0s27kb9WMWeokv3+9cHysh+M3U98NlE1Zr+XUBALAWC1V0uIF6
KJJuQ/s4PV1mminoVX5rihlc1G0aG23o43tWmfpXVkjvs/CmhSFoiIRvoHCKSCyT
kaifilmz+3bqQJTQiQIDAQAB
-----END PUBLIC KEY-----
```



### 二:具体代码

```go
package rsaDemo

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"log"
	"os"
	"testing"
)

// 用来保存公钥和私钥
var (
	publicKey  []byte
	privateKey []byte
)

// 读取密钥文件,返回密钥
func ReadFile(src string) []byte {
	open, err := os.Open(src)
	if err != nil {
		log.Println(err)
		return nil
	}
	bytes := make([]byte, 4096)
	n, err := open.Read(bytes)
	if err != nil {
		log.Println(err)
		return nil
	}
	return bytes[:n]
}

// 读取公钥和私钥
func ReadRSAKey(publicKeySrc, privateKeySrc string) {
	publicKey = ReadFile(publicKeySrc)
	privateKey = ReadFile(privateKeySrc)
}

// 加密
func RSAEncrypt(origData []byte) []byte {
  // 解密pem格式的公钥
	block, _ := pem.Decode(publicKey)
	if block == nil {
		return nil
	}
  // 解析公钥，目前数字证书一般都是基于ITU（国际电信联盟）指定的x.509标准
	pkixPublicKey, err := x509.ParsePKIXPublicKey(block.Bytes)
	if err != nil {
		log.Println(err)
		return nil
	}
  // 类型断言
	pub := pkixPublicKey.(*rsa.PublicKey)
  // 加密明文
	encryptPKCS1v15, err := rsa.EncryptPKCS1v15(rand.Reader, pub, origData)
	if err != nil {
		log.Println(err)
		return nil
	}
	return encryptPKCS1v15
}

// 解密
func RSADecrypt(cipherText []byte) []byte {
  // 解密
	block, _ := pem.Decode(privateKey)
	if block == nil {
		return nil
	}
  // 解析PKCS1格式的私钥
	pkcs1PrivateKey, err := x509.ParsePKCS1PrivateKey(block.Bytes)
	if err != nil {
		log.Println(err)
		return nil
	}
  // 解密密文
	pkcs1v15, err := rsa.DecryptPKCS1v15(rand.Reader, pkcs1PrivateKey, cipherText)
	if err != nil {
		log.Println(err)
		return nil
	}
	return pkcs1v15
}

// 测试加密和解密
func TestRSA(t *testing.T) {
	ReadRSAKey("./rsa_public_key.pem", "./rsa_private_key.pem")
	plain := "hello world"
	encrypt := RSAEncrypt([]byte(plain))
	fmt.Println("密文: ", encrypt)
	decrypt := RSADecrypt(encrypt)
	fmt.Println("明文: ", string(decrypt))
}
==================================
=== RUN   TestRSA
密文:  [34 64 55 21 154 211 137 229 67 104 102 50 120 155 96 8 155 113 119 184 197 242 49 246 15 55 123 155 50 183 242 212 214 111 74 84 51 53 35 127 135 39 173 208 68 21 117 242 86 142 186 57 197 74 166 69 20 23 212 87 85 203 245 80 237 182 50 168 21 63 251 76 65 238 43 252 203 59 163 197 183 70 59 235 141 241 148 153 101 125 123 190 128 103 69 109 153 49 171 61 128 141 98 188 69 120 93 64 192 3 81 236 148 159 24 121 208 24 49 163 246 142 188 224 83 103 187 200]
明文:  hello world
```



## 实现js前端加密和go后端解密

``使用jsencrypt进行加密``

>jsencrypt默认会把加密后的数据进行base64编码,所以go后端需要先进行base64的解码后才能解密

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jsencrypt/3.3.2/jsencrypt.min.js"></script>
</head>

<body>

</body>
<script>
  // 这里必须是pem文件的全部,包括头和尾数据
    pk = '-----BEGIN PUBLIC KEY-----MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5vgoY3FkQmR+BM76mFHx4odRPu5Qlz4cYXP1jxY0s27kb9WMWeokv3+9cHysh+M3U98NlE1Zr+XUBALAWC1V0uIF6KJJuQ/s4PV1mminoVX5rihlc1G0aG23o43tWmfpXVkjvs/CmhSFoiIRvoHCKSCyTkaifilmz+3bqQJTQiQIDAQAB-----END PUBLIC KEY-----'
    var encrypt = new JSEncrypt();
    encrypt.setPublicKey(pk)
    var encodeData = encrypt.encrypt("hello world");
    console.log(encodeData);
</script>

</html>
```

```go
func TestRSA(t *testing.T) {
  // base64解码	
  decodeString, err := base64.StdEncoding.DecodeString("js里面的encodeData数据")
  if err != nil {
    log.Println(err)
    return
  }
  // 配合上边go代码的解密即可实现
  decrypt := RSADecrypt(decodeString)
  fmt.Println("明文: ", string(decrypt))
}
============
明文:  hello world
```

