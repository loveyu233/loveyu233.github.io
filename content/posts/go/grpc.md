---
title: "go GRPC使用"
date: 2021-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- grpc
---

# jsonrpc

>把jsonrpc.ServeConn(conn)改为grpc.ServeConn(conn)就是grpc了
>
>``为什么用jsongrpc？``
>
>因为grpc使用的为go语言独有的数据序列化gob，导致其他语言无法解析；
>
>使用jsonrpc的话数据序列化就使用json这样其他语言都可以解析使用

## 服务端

```go
type user struct {
}

func (u user) Hello(name string, res *string) error {
   *res = name + "  over!"
   return nil
}
func main() {
   // 注册服务
   err := rpc.RegisterName("hello", new(user))
   if err != nil {
      fmt.Println("rpc.RegisterName err", err)
      return
   }
   // 启动监听
   listen, err := net.Listen("tcp", "localhost:8888")
   if err != nil {
      fmt.Println("net.Listen err", err)
      return
   }
   for true {
      // 建立连接
      conn, err := listen.Accept()
      if err != nil {
         fmt.Println("listen.Accept err", err)
         return
      }
      // 绑定服务
      go jsonrpc.ServeConn(conn)
   }
}
```

## 客户端

```go
func main() {
   // 连接
   dial, err := jsonrpc.Dial("tcp", "localhost:8888")
   if err != nil {
      fmt.Println("jsonrpc.Dial", err)
      return
   }
   var str string
   // 调用
   err = dial.Call("hello.Hello", "zz", &str)
   if err != nil {
      fmt.Println("dial.Call err", err)
   }
   fmt.Println("aaaa:", str)
}
```



## 缺点

>这样写代码的话会导致代码量太高，注册服务不好查看

## 改进

>使用protobuf来编写服务



# protobuf

> 谷歌出的一种数据序列化语言，和其他数据序列化语言比较，protobuf简单、数据之间传输更小、更快

## 语法

>详细语法百度查看

```protobuf
// 声明版本
syntax = "proto3";

// 包
package pb;

//  生成go语言所在包
option go_package = "../pb";

// 相当于struct
message Tea {
  int32 age = 1;
  string name = 2;
}

//  服务
service HelloWorld {
  rpc Test1(Tea) returns (Tea);
}
```



## 命令

### proto编译grpc

```shell
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    ./*.proto
```



### proto编译getaway

```go
protoc \
  --go_out . --go_opt paths=source_relative \
  --go-grpc_out . --go-grpc_opt paths=source_relative \
  --grpc-gateway_out . --grpc-gateway_opt paths=source_relative \
  ./proto/rest.proto
```



# protoGRPC

## 入门篇

### .proto

>使用命令编译为go文件。编译命令看上文

```protobuf
syntax = "proto3";

package pb;

option go_package = "../pb";

message Tea {
  int32 age = 1;
  string name = 2;
}

service HelloWorld {
  rpc Test1(Tea) returns (Tea);
}
```



### server.go

```go
type stu struct {
}

/* 实现pb下的服务端接口
type HelloWorldServer interface {
	Test1(context.Context, *Tea) (*Tea, error)
}
*/

func (s stu) Test1(ctx context.Context, tea *pb.Tea) (*pb.Tea, error) {
	tea.Name = "hello"
	tea.Age = 66
	return tea, nil
}

func main() {
  //	创建服务
	server := grpc.NewServer()
  //	注册服务
	pb.RegisterHelloWorldServer(server, new(stu))
  //	启动监听
	listen, err := net.Listen("tcp", "localhost:9999")
	if err != nil {
		fmt.Println(err)
	}
  //	启动服务
	server.Serve(listen)
}
```



### client.go

>客户端不需要实现接口
>
>grpc.WithInsecure()：后边会解决这个问题

```go
func main() {
   //	连接
   dial, err := grpc.Dial("localhost:9999", grpc.WithInsecure())
   if err != nil {
      fmt.Println(err)
   }
   //	创建客户端
   client := pb.NewHelloWorldClient(dial)
   tea := new(pb.Tea)
   tea.Name = "abc"
   //	调用服务
   t, err := client.Test1(context.TODO(), tea)
   if err != nil {
      fmt.Println(err)
   }
   //	输出返回值
   fmt.Println(t.Name, "-->", t.Age)
}
```



## 进阶篇

### .proto

>流式传输，相当于一个不断开的连接可以一直传输数据

```protobuf
syntax = "proto3";

package pb;

option go_package = "./proto";

message Req {
  string message = 1;
}

message Res {
  string message = 1;
}

service HelloGRPC {
  rpc SayHi(Req) returns (Res);
  //	输入流
  rpc SayHiIn(stream Req) returns (Res);
  //	输出流
  rpc SayHiOut(Req) returns (stream Res);
  //	输入输出流
  rpc SayHiIO(stream Req) returns (stream Res);
}
```

### server.go/client.go

>服务端注册

```go
func main() {
   listen, err := net.Listen("tcp", ":8888")
   if err != nil {
      fmt.Println("注册失败", err.Error())
      return
   }
   defer listen.Close()
   newServer := grpc.NewServer()
   pb.RegisterHelloGRPCServer(newServer, &server{})
   newServer.Serve(listen)
   reflection.Register(newServer)
}
```

>客户端调用

```go
func main() {
   conn, err := grpc.Dial(":8888", grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
      fmt.Println("连接失败", err.Error())
      return
   }
   defer conn.Close()
   client := pb.NewHelloGRPCClient(conn)
   //sayHi(client)
   //sayHiIn(client)
   //sayHiOut(client)
   sayHiIO(client)
}
```



>服务端实现接口

```go
/*
实现pb的服务端接口
type HelloGRPCServer interface {
	SayHi(context.Context, *Req) (*Res, error)
	SayHiIn(HelloGRPC_SayHiInServer) error
	SayHiOut(*Req, HelloGRPC_SayHiOutServer) error
	SayHiIO(HelloGRPC_SayHiIOServer) error
	mustEmbedUnimplementedHelloGRPCServer()
}
*/
type server struct {
   pb.UnimplementedHelloGRPCServer
}
```

>客户端不需要实现



#### 普通传输

>服务端

```go
func (s server) SayHi(ctx context.Context, req *pb.Req) (*pb.Res, error) {
   message := req.GetMessage()
   fmt.Println(message)
   return &pb.Res{Message: fmt.Sprintf("收到消息:%s", message)}, nil
}
```

>客户端

```go
func sayHi(client pb.HelloGRPCClient) {
  
   hi, err := client.SayHi(context.Background(), &pb.Req{Message: "client"})
   if err != nil {
      fmt.Println("调用失败", err.Error())
      return
   }
   fmt.Println(hi.GetMessage())
  
}
```

>``这里的传入传出是相对于服务端来说，传入即位入参、传出即位出参``

#### 流式传入

>客户端一直向服务端传输，当服务端向客户端传输时即为结束此次连接
>
>``服务端：``
>
>in, err := client.SayHiIn(context.Background())：获取客户端流
>
>in.Send(&pb.Req{Message: fmt.Sprintf("%s:%d", "message", i)})：客户端发送数据，参数为发送的数据
>
>``客户端：``
>
>inServer.Recv()：持续发送数据，参数为发送的数据
>
>inServer.SendAndClose(&pb.Res{Message: "结束"})：结束此次连接并发送数据
>
>recv.GetMessage()：获取服务端传回的数据

>服务端

```go
func (s server) SayHiIn(inServer pb.HelloGRPC_SayHiInServer) error {
   for {
      recv, err := inServer.Recv()
      if err != nil {
         fmt.Println(err)
         inServer.SendAndClose(&pb.Res{Message: "结束"})
         break
      }
      fmt.Println("收到消息:", recv.GetMessage())
   }
   return nil
}
```

>客户端

```go
func sayHiIn(client pb.HelloGRPCClient) {
   in, err := client.SayHiIn(context.Background())
   if err != nil {
      fmt.Println(err.Error())
   }
   for i := 0; i < 10; i++ {
      err := in.Send(&pb.Req{Message: fmt.Sprintf("%s:%d", "message", i)})
      if err != nil {
         fmt.Println(err.Error(), 26)
         break
      }
   }
   recv, err := in.CloseAndRecv()
   if err != nil {
      fmt.Println(err.Error())
   }
   fmt.Println(recv.GetMessage())
}
```



#### 流式传出

>服务端一直向客户端发送数据。由客户端先发送数据，服务端接收到数据然后进行流式的传出数据
>
>``客户端：``
>
>out, err := client.SayHiOut(context.Background(), &pb.Req{Message: "233"})：客户端发送数据，获取返回值
>
>recv, err := out.Recv()：通过循环来只需获得数据，当有err时表明服务端结束此次连接
>
>recv.GetMessage()：获取服务端的数据
>
>``服务端：``
>
>message := req.GetMessage()：获取客户端的数据
>
>outServer.Send()：可以循环向客户端发送数据，参数为发送的数据

>服务端

```go

func (s server) SayHiOut(req *pb.Req, outServer pb.HelloGRPC_SayHiOutServer) error {
   message := req.GetMessage()
   for i := 0; i < 10; i++ {
      outServer.Send(&pb.Res{Message: fmt.Sprintf("%s:%d", message, i)})
   }
   return nil
}
```

>客户端

```go
func sayHiOut(client pb.HelloGRPCClient) {
   out, err := client.SayHiOut(context.Background(), &pb.Req{Message: "233"})
   if err != nil {
      fmt.Println(err.Error())
   }
   for true {
      recv, err := out.Recv()
      if err != nil {
         fmt.Println(err.Error())
         break
      }
      fmt.Println(recv.GetMessage())
   }
}
```



#### 流式传入传出

>客户端和服务端之间持续的传入传出数据
>
>``服务端：``
>
>ioServer.Recv()：获取客户端流
>
>recv.GetMessage()：或取客户端流的数据
>
>ioServer.Send(&pb.Res{Message: fmt.Sprintf("get meg:%s", s)})：向客户端发送数据
>
>``客户端：``
>
>io, err := client.SayHiIO(context.Background())：获取io
>
>io.Send(&pb.Req{Message: "client"})：向服务端发送数据
>
>recv, err := io.Recv()：获取服务端流
>
>recv.GetMessage()：获取服务端流数据

>服务端

```go

func (s server) SayHiIO(ioServer pb.HelloGRPC_SayHiIOServer) error {
   strings := make(chan string)
   go func() {
      i := 1
      for true {
         recv, err := ioServer.Recv()
         if i >= 10 {
            strings <- "END"
            break
         }
         if err != nil {
            fmt.Println(err.Error())
            strings <- "END"
            break
         }
         strings <- recv.GetMessage()
         i++
      }
   }()
   for true {
      s := <-strings
      if s == "END" {
         break
      }
      ioServer.Send(&pb.Res{Message: fmt.Sprintf("get meg:%s", s)})
   }
   return nil
}
```

>客户端

```go
func sayHiIO(client pb.HelloGRPCClient) {
   io, err := client.SayHiIO(context.Background())
   if err != nil {
      fmt.Println(err.Error())
   }
   group := sync.WaitGroup{}
   group.Add(2)
   go func() {
      for {
         err := io.Send(&pb.Req{Message: "client"})
         if err != nil {
            fmt.Println(err, 61)
            group.Done()
            break
         }
      }
   }()
   go func() {
      for true {
         recv, err := io.Recv()
         if err != nil {
            fmt.Println(err, 71)
            group.Done()
            break
         }
         fmt.Println(recv.GetMessage())
      }
   }()
   group.Wait()
   fmt.Println("结束")
}
```



### 完整代码

#### server.go

```go
package main

import (
   "context"
   "fmt"
   "google.golang.org/grpc"
   "google.golang.org/grpc/reflection"
   "net"
   "protoDemo/sources/pb"
)

type server struct {
   pb.UnimplementedHelloGRPCServer
}

func (s server) SayHi(ctx context.Context, req *pb.Req) (*pb.Res, error) {
   message := req.GetMessage()
   fmt.Println(message)
   return &pb.Res{Message: fmt.Sprintf("收到消息:%s", message)}, nil
}

func (s server) SayHiIn(inServer pb.HelloGRPC_SayHiInServer) error {
   for {
      recv, err := inServer.Recv()
      if err != nil {
         fmt.Println(err)
         inServer.SendAndClose(&pb.Res{Message: "结束"})
         break
      }
      fmt.Println("收到消息:", recv.GetMessage())
   }
   return nil
}

func (s server) SayHiOut(req *pb.Req, outServer pb.HelloGRPC_SayHiOutServer) error {
   message := req.GetMessage()
   for i := 0; i < 10; i++ {
      outServer.Send(&pb.Res{Message: fmt.Sprintf("%s:%d", message, i)})
   }
   return nil
}

func (s server) SayHiIO(ioServer pb.HelloGRPC_SayHiIOServer) error {
   strings := make(chan string)
   go func() {
      i := 1
      for true {
         recv, err := ioServer.Recv()
         if i >= 10 {
            strings <- "END"
            break
         }
         if err != nil {
            fmt.Println(err.Error())
            strings <- "END"
            break
         }
         strings <- recv.GetMessage()
         i++
      }
   }()
   for true {
      s := <-strings
      if s == "END" {
         break
      }
      ioServer.Send(&pb.Res{Message: fmt.Sprintf("get meg:%s", s)})
   }
   return nil
}

func main() {
   listen, err := net.Listen("tcp", ":8888")
   if err != nil {
      fmt.Println("注册失败", err.Error())
      return
   }
   defer listen.Close()
   newServer := grpc.NewServer()
   pb.RegisterHelloGRPCServer(newServer, &server{})
   newServer.Serve(listen)
   reflection.Register(newServer)
}
```

#### client.go

```go
package main

import (
   "context"
   "fmt"
   "google.golang.org/grpc"
   "google.golang.org/grpc/credentials/insecure"
   "protoDemo/sources/pb"
   "sync"
)

func sayHi(client pb.HelloGRPCClient) {
   hi, err := client.SayHi(context.Background(), &pb.Req{Message: "client"})
   if err != nil {
      fmt.Println("调用失败", err.Error())
      return
   }
   fmt.Println(hi.GetMessage())
}
func sayHiIn(client pb.HelloGRPCClient) {
   in, err := client.SayHiIn(context.Background())
   if err != nil {
      fmt.Println(err.Error())
   }
   for i := 0; i < 10; i++ {
      err := in.Send(&pb.Req{Message: fmt.Sprintf("%s:%d", "message", i)})
      if err != nil {
         fmt.Println(err.Error(), 26)
         break
      }
   }
   recv, err := in.CloseAndRecv()
   if err != nil {
      fmt.Println(err.Error())
   }
   fmt.Println(recv.GetMessage())
}
func sayHiOut(client pb.HelloGRPCClient) {
   out, err := client.SayHiOut(context.Background(), &pb.Req{Message: "233"})
   if err != nil {
      fmt.Println(err.Error())
   }
   for true {
      recv, err := out.Recv()
      if err != nil {
         fmt.Println(err.Error())
         break
      }
      fmt.Println(recv.GetMessage())
   }
}
func sayHiIO(client pb.HelloGRPCClient) {
   io, err := client.SayHiIO(context.Background())
   if err != nil {
      fmt.Println(err.Error())
   }
   group := sync.WaitGroup{}
   group.Add(2)
   go func() {
      for {
         err := io.Send(&pb.Req{Message: "client"})
         if err != nil {
            fmt.Println(err, 61)
            group.Done()
            break
         }
      }
   }()
   go func() {
      for true {
         recv, err := io.Recv()
         if err != nil {
            fmt.Println(err, 71)
            group.Done()
            break
         }
         fmt.Println(recv.GetMessage())
      }
   }()
   group.Wait()
   fmt.Println("结束")
}
func main() {
   conn, err := grpc.Dial(":8888", grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
      fmt.Println("连接失败", err.Error())
      return
   }
   defer conn.Close()
   client := pb.NewHelloGRPCClient(conn)
   //sayHi(client)
   //sayHiIn(client)
   //sayHiOut(client)
   sayHiIO(client)
}
```



## getaway篇

>``为什么需要getaway？``
>
>前边的服务调用都只能通过rpc调用，不能使用http调用。而getaway就是用来解决http调用rpc服务的技术
>
>``使用前提``
>
>需要两个proot文件：annotations.proto和http.proto



### .proto

>编译为go文件，编译命令看上边

```protobuf
syntax = "proto3";

package template;

option go_package = "./proto";

//	引入
import "google/api/annotations.proto";

message Req {
  string username = 1;
  string password = 2;
}

message Res {
  bool msg = 3;
}

service HelloServer {
  rpc SayHi(Req) returns(Res) {
    option (google.api.http) = {
    
    //	get请求
      get: "/user"
    };
  }
}
```



### server.go

>一样需要实现接口
>
>```go
>type HelloServerServer interface {
>SayHi(context.Context, *Req) (*Res, error)
>mustEmbedUnimplementedHelloServerServer()
>}
>```

```go
package main

type HelloServer struct {
   proto.UnimplementedHelloServerServer
}

func (h HelloServer) SayHi(ctx context.Context, req *proto.Req) (*proto.Res, error) {
   if req.GetUsername() == "a" && req.GetPassword() == "b" {
      return &proto.Res{Msg: true}, nil
   } else {
      return &proto.Res{Msg: false}, nil
   }
}
```

>启动http服务

```go
func runGrpcServer(s *sync.WaitGroup) {
   server := grpc.NewServer()
   proto.RegisterHelloServerServer(server, &HelloServer{})
   listen, err := net.Listen("tcp", ":8080")
   if err != nil {
      s.Done()
      return
   }
   err = server.Serve(listen)
   if err != nil {
      s.Done()
      return
   }
   s.Done()
}
```

>启动rpc服务

```go
func runHttpServer(s *sync.WaitGroup) {
    conn, _ := grpc.DialContext(
      context.Background(),
      ":9090",
      grpc.WithBlock(),
      grpc.WithInsecure(),
    )

    mux := runtime.NewServeMux()
    // 注册服务
    proto.RegisterHelloServerHandler(context.Background(), mux, conn)

    server := &http.Server{
      Addr:    ":8090",
      Handler: mux,
    }

    server.ListenAndServe()
    s.Done()
}
```

>goroutine启动rpc服务和http服务

```go
func main() {
   group := sync.WaitGroup{}
   group.Add(2)
   //	使用go 来运行两个服务
   go runGrpcServer(&group)
   go runHttpServer(&group)
   group.Wait()
}
```



### client.go

>可以直接通过浏览器调用



## 实战篇

#### 生成证书

> 查看生成证书篇。共需要证书文件：

ca.pem; client.key; client.pem

ca.pem; server.key; server.pem



#### 使用证书

> 大体上不需要改变，只需要添加证书即可

##### server.go

```go
func main() {
	cert, _ := tls.LoadX509KeyPair("helloWorld/certServer/server.pem", "helloWorld/certServer/server.key")
	pool := x509.NewCertPool()
	ca, _ := ioutil.ReadFile("helloWorld/certServer/ca.pem")
	pool.AppendCertsFromPEM(ca)
	creeds := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{cert},
		ClientAuth:   tls.RequestClientCert,
		ClientCAs:    pool})
	listen, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println(err)
		return
	}
	ser := grpc.NewServer(grpc.Creds(creeds))
	proto.RegisterHelloServerServer(ser, &HelloServer{})
	reflection.Register(ser)
	ser.Serve(listen)
}
```

##### client.go

```go
func main() {
   cert, _ := tls.LoadX509KeyPair("helloWorld/certClient/client.pem", "helloWorld/certClient/client.key")
   pool := x509.NewCertPool()
   ca, _ := ioutil.ReadFile("helloWorld/certClient/ca.pem")
   pool.AppendCertsFromPEM(ca)
   creeds := credentials.NewTLS(&tls.Config{
      Certificates: []tls.Certificate{cert},
      ServerName:   "localhost",
      RootCAs:      pool})

   dial, err := grpc.Dial(":8080", grpc.WithTransportCredentials(creeds))
   if err != nil {
      fmt.Println(err)
   }
   defer dial.Close()
   client := proto.NewHelloServerClient(dial)
   req, err := client.SayHi(context.Background(), &proto.Req{Username: "a", Password: "b"})
   if err != nil {
      fmt.Println(err)
   }
   fmt.Println(req.GetMsg())
}
```

##### gateway.go

```go
package main

import (
   "context"
   "crypto/tls"
   "crypto/x509"
   "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
   "google.golang.org/grpc"
   "google.golang.org/grpc/credentials"
   "io/ioutil"
   "net"
   "net/http"
   "protoDemo/gateway/proto"
   "sync"
)

type HelloServer struct {
   proto.UnimplementedHelloServerServer
}

func (h HelloServer) SayHi(ctx context.Context, req *proto.Req) (*proto.Res, error) {
   if req.GetUsername() == "a" && req.GetPassword() == "b" {
      return &proto.Res{Msg: true}, nil
   } else {
      return &proto.Res{Msg: false}, nil
   }
}

func getServerCert() credentials.TransportCredentials {
   cert, _ := tls.LoadX509KeyPair("helloWorld/certServer/server.pem", "helloWorld/certServer/server.key")
   pool := x509.NewCertPool()
   ca, _ := ioutil.ReadFile("helloWorld/certServer/ca.pem")
   pool.AppendCertsFromPEM(ca)
   creeds := credentials.NewTLS(&tls.Config{
      Certificates: []tls.Certificate{cert},
      ClientAuth:   tls.RequestClientCert,
      ClientCAs:    pool})
   return creeds
}

func getClientCert() credentials.TransportCredentials {
   cert, _ := tls.LoadX509KeyPair("helloWorld/certClient/client.pem", "helloWorld/certClient/client.key")
   pool := x509.NewCertPool()
   ca, _ := ioutil.ReadFile("helloWorld/certClient/ca.pem")
   pool.AppendCertsFromPEM(ca)
   creeds := credentials.NewTLS(&tls.Config{
      Certificates: []tls.Certificate{cert},
      ServerName:   "localhost",
      RootCAs:      pool})
   return creeds

}

func runGrpcServer(s *sync.WaitGroup) {
   server := grpc.NewServer(grpc.Creds(getServerCert()))
   proto.RegisterHelloServerServer(server, &HelloServer{})
   listen, err := net.Listen("tcp", ":8080")
   if err != nil {
      s.Done()
      return
   }
   err = server.Serve(listen)
   if err != nil {
      s.Done()
      return
   }
   s.Done()
}
func runHttpServer(s *sync.WaitGroup) {
   mux := runtime.NewServeMux()
   options := []grpc.DialOption{grpc.WithTransportCredentials(getClientCert())}
   err := proto.RegisterHelloServerHandlerFromEndpoint(
      context.Background(),
      mux,
      "localhost:8080",
      options,
   )
   if err != nil {
      s.Done()
   }
   httpServer := http.Server{
      Addr:    ":80",
      Handler: mux,
   }
   err = httpServer.ListenAndServe()
   if err != nil {
      s.Done()
      return
   }
   s.Done()
}

func main() {
   group := sync.WaitGroup{}
   group.Add(2)
   go runGrpcServer(&group)
   go runHttpServer(&group)
   group.Wait()
}
```
