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

>分配字段编号说明:
>
>消息定义中的每个字段都有一个唯一的编号。这些字段编号用于在 消息二进制格式中标识您的字段，并且在使用消息类型后不应更改。
>[1, 15]之内的标识号在编码的时候会占用一个字节。[16, 2047]之内的标识号则占用2个字节。
>最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的[19000－19999],因为是预留信息，如果使用，编译时会报错。
>
>``singular``：消息中至多存在一个该字段的数据。使用 proto3 语法时，当没有为给定字段指定其他字段规则时，这是默认字段规则。
>``optional``：与 singular 类似，不同之处在于可以检查该值是否已经显式设置了值。字段有两种可能的状态:
>该字段已设置，并包含从连接中显式设置或解析的值。它将被序列化到连接上。
>该字段未设置，将返回默认值。它不会被序列化。
>``repeated``：该字段类型可以在消息中可以重复设置多次，重复值的顺序将被保留。（设置成为数组类型）
>``map``：成对的键/值字段类型。
>
>`` reserved``:保留标识符, 对于特殊的字段名或者编号通过完全删除字段或将其注释掉来更新消息类型，如果后面出现其他用户对该消息进行更新重用了特殊的字段名或者编号，可能会导致严重的错误，包括数据损坏、出现隐私漏洞等。
>为了确保这种情况不会发生的一种方法就是用保留标识符指定保留已删除的字段名或者编号。如果有其他用户试图重用这些字段名或编号，protobuf则会报错预警。

```protobuf
// 指明当前使用proto3语法，如果不指定，编译器会使用proto2
syntax = "proto3";

// package声明符，用来防止消息类型有命名冲突
package msg;

// 选项信息，对应go的包路径
option go_package = "server/msg";

// message关键字，像go中的结构体
message FirstMsg {
  // 类型 字段名 标识号
  int32 id = 1;
  string name=2;
  string age=3;
}

// 在这个消息中标记
message DemoMsg {
  // 标示号：1，2，10，11，12，13 都不能用
  reserved 1, 2, 10 to 13;
  // 字段名 test、name 不能用
  reserved "test","name";
  // 不能使用字段名，提示:Field name 'name' is reserved
  string name = 3;
  // 不能使用标示号,提示:Field 'id' uses reserved number 11
  int32 id = 11;
}

// 另外一个消息还是可以正常使用
message Demo2Msg {
  // 标示号可以正常使用
  int32 id = 1;
  // 字段名可以正常使用
  string name = 2;
}
```



### 枚举

>每个枚举类型必须将其第一个类型映射为编号0, 原因有两个：
>
>1. 必须有一个零值，以便我们可以使用 0 作为数字 默认值。
>2. 零值必须是第一个元素，以便与第一个枚举值始终为默认值的proto2语义兼容 。
>
>枚举：在定义消息类型时，希望其中一个字段只是预定义值列表中的一个值。

```go
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  Corpus corpus = 4;
}
```



### 引入其他proto文件

```go
// 导入其他proto文件
import "proto/class.proto";
```



### map类型

>注意：
>
>key_type只能是任何整数或字符串类型(除浮点类型和任何标量bytes类型)。
>enum 不能作为key_type和value_type定义的类型。
>map字段不能是repeated。

```go
map<key_type, value_type> map_field = N;
```



### 切片/数组类型

```go
// repeated允许字段重复，对于Go语言来说，它会编译成数组(slice of type)类型的格式
message DemoSliceMsg {
  // 会生成 []int32
  repeated int32 id = 1;
  // 会生成 []string
  repeated string name = 2;
  // 会生成 []float32
  repeated float price = 3;
  // 会生成 []float64
  repeated double money = 4;
}
```







![dy](https://www.loveyu.asia//img/dy.png)





## 命令

>快速生成命令: https://github.com/loveyu233/makefileInit

### proto编译grpc

```shell
protoc --go_out=. --go-grpc_out=. proto文件地址
`` protoc --go_out=. --go-grpc_out=. pb/user.proto ``
```



### proto编译getaway

```go
protoc \
  --go_out . --go_opt paths=source_relative \
  --go-grpc_out . --go-grpc_opt paths=source_relative \
  --grpc-gateway_out . --grpc-gateway_opt paths=source_relative \
  ./proto/rest.proto
```



## 序列化和反序列化

```go
	ms := pb.HelloWorld{Msg: "helloWorld"}
	marshal, err := proto.Marshal(&ms)
	if err != nil {
		t.Log(err)
		return
	}
	fmt.Println(marshal)
	unMs := pb.HelloWorld{}
	err = proto.Unmarshal(marshal, &unMs)
	fmt.Println(unMs.Msg)
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



## 证书验证篇

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



## 拦截器篇

>GRPC的拦截器类似于HTTP服务的中间件,但又不同于中间件,拦截器既可以在client端也可以在server端,且GRPC的拦截器共两种类型,分别为:``一元 流式`` 一元既一次请求一次回复,流式为建立一次连接后可以通过这次连接发送多次数据.这两种类型的拦截器可以作用于client和server所以共有``四种拦截器`` 
>
>``所谓的拦截器其实就是一个函数，可以分为预处理(pre-processing)、调用RPC方法(invoking RPC method)、后处理(post-processing)三个阶段。`` 
>
>client使用:
>
>``dial, err := grpc.Dial("127.0.0.1:8888",
>    grpc.WithInsecure(),
>    grpc.WithUnaryInterceptor(UnaryInterceptor),
>    grpc.WithStreamInterceptor(StreamInterceptor),
>)`` 
>
>server使用:
>
>``server := grpc.NewServer(
>    grpc.UnaryInterceptor(UnaryInterceptor),
>    grpc.StreamInterceptor(StreamInterceptor),
>)`` 

### Client一元拦截器

>client拦截器主要用于发送请求前添加元数据,例如认证数据等...
>
>ctx：Go语言中的上下文，一般和 Goroutine 配合使用，起到超时控制的效果
>method：当前调用的 RPC 方法名
>req：本次请求的参数，只有在处理前阶段修改才有效
>reply：本次请求响应，需要在处理后阶段才能获取到
>cc：gRPC 连接信息
>invoker：可以看做是当前 RPC 方法，一般在拦截器中调用 invoker 能达到调用 RPC 方法的效果，当然底层也是 gRPC 在处理。
>opts：本次调用指定的 options 信息

```go
func UnaryInterceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	// 调用前执行
	log.Println("远程调用的方法:", method)
	log.Println("客户端发送的数据:", req)
	// 添加元数据,参数为keyValue可以添加多个, 切记返回值赋值为ctx上下文
	ctx = metadata.AppendToOutgoingContext(ctx, "token", "1234abcd", "msg", "test")
	// invoker 发起调用
	err := invoker(ctx, method, req, reply, cc, opts...)
	// 调用后执行
	if err != nil {
		panic(err)
	}
	log.Println("服务器回复的数据:", reply)
	return err
}
```

```shell
远程调用的方法: /pb.Operate/SayHi
客户端发送的数据: msg:"hello world"
服务器回复的数据: msg:"hello world over! "
```



### Server一元拦截器

>可以用来校验请求等...
>
>ctx：请求上下文
>req：RPC 方法的请求参数
>info：RPC 方法的所有信息
>handler：RPC 方法真正执行的逻辑

```go
func UnaryInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
	log.Println("req:", req)
	log.Println("info:", info.FullMethod)
	log.Println("info:", info.Server)
	// 解析客户端携带数据
	md, b := metadata.FromIncomingContext(ctx)
	if !b {
		return nil, errors.New("meta数据读取错误")
	}
	// 获取客户端携带的数据
	token := md.Get("token")
	msg := md.Get("msg")
	log.Println("token:", token)
	log.Println("msg:", msg)
	//
	p, ok := peer.FromContext(ctx)
	if !ok {
		return nil, errors.New("ip数据读取错误")
	}
	ip := p.Addr.String()
	log.Println("ip:", ip)
	log.Println("p.AuthInfo.AuthType(): ", p.AuthInfo.AuthType())
	// 处理请求
	response, err := handler(ctx, req)
	if err != nil {
		panic(err)
	}
	// 返回处理后的数据
	return response, err
}
```

```shell
 req: msg:"hello world"
 info: /pb.Operate/SayHi
 info: &{{}}
 token: [1234abcd]
 msg: [test]
 ip: 127.0.0.1:55952
```



### Client流式拦截器

>1. 声明结构体
>2. 实现``RecvMsg`` ``SendMsg``
>3. 发送请求``stream, err := streamer(ctx, desc, cc, method, opts...)``
>4. 使用自定义``newStreamClient(stream), err`` 
>5. 发送和 接收的数据都会经过自定义实现的``RecvMsg和SendMsg`` 可以在这两个方法中做数据处理

```go
func StreamInterceptor(ctx context.Context, desc *grpc.StreamDesc, cc *grpc.ClientConn, method string, streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
	stream, err := streamer(ctx, desc, cc, method, opts...)
	if err != nil {
		panic(err)
	}
	return newStreamClient(stream), err
}

type StreamClient struct {
	grpc.ClientStream
}

// 对ClientStream进行包装
func newStreamClient(c grpc.ClientStream) grpc.ClientStream {
	return &StreamClient{c}
}

// 可以对发送的消息和传输的消息进行操作
func (s StreamClient) RecvMsg(m interface{}) error {
	return s.ClientStream.RecvMsg(m)
}

func (s StreamClient) SendMsg(m interface{}) error {
	return s.ClientStream.SendMsg(m)
}
```



### Server流式拦截器

>和client基本一致

```go
type StreamServer struct {
	grpc.ServerStream
}

func newStreamServer(s grpc.ServerStream) *StreamServer {
	return &StreamServer{s}
}

func (s StreamServer) RecvMsg(m interface{}) error {
	log.Println("RecvMsg: ", m)
	return s.ServerStream.RecvMsg(m)
}

func (s StreamServer) SendMsg(m interface{}) error {
	log.Println("SendMsg: ", m)
	return s.ServerStream.SendMsg(m)
}
func StreamInterceptor(srv interface{}, ss grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
	server := newStreamServer(ss)
	return handler(srv, server)
}
```



## 实现多个拦截器

```go
// 服务端
grpc.ChainUnaryInterceptor(多个拦截器)
grpc.ChainStreamInterceptor(多个拦截器)

// 客户端
grpc.WithChainUnaryInterceptor(多个拦截器)
grpc.WithChainStreamInterceptor(多个拦截器)
```



# 使用etcd服务注册和发现

## Server

```go
package main

import (
	"context"
	"etcd-demo/pb"
	"fmt"
	eclient "go.etcd.io/etcd/client/v3"
	"go.etcd.io/etcd/client/v3/naming/endpoints"
	"google.golang.org/grpc"
	"net"
	"time"
)

type Demo struct {
	pb.UnimplementedTestDemoServer
}

func (d Demo) Demo1(ctx context.Context, in *pb.Req) (*pb.Resp, error) {
	fmt.Println(in.Msg)
	return &pb.Resp{Msg: in.Msg + "hello 9001"}, nil
}

type User struct {
	pb.UnimplementedUserSerServer
}

func (u User) Login(ctx context.Context, in *pb.User) (*pb.UserResp, error) {
	resp := &pb.UserResp{}
	if in.Username == "a" && in.Password == "b" {
		resp.Status = 200
	}
	resp.Status = 500
	return resp, nil
}

const (
	// grpc 服务名
	MyService = "xiaoxu/demo"
	// etcd 端口
	MyEtcdURL = "http://localhost:2379"

	ServiceAddr = "127.0.0.1:9002"
)

func main() {
	server := grpc.NewServer()
	listen, err := net.Listen("tcp", ServiceAddr)
	if err != nil {
		panic(err)
	}
	pb.RegisterTestDemoServer(server, &Demo{})
	pb.RegisterUserSerServer(server, &User{})

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	// 注册 grpc 服务节点到 etcd 中
	go registerEndPointToEtcd(ctx, ServiceAddr)

	err = server.Serve(listen)
	if err != nil {
		panic(err)
	}
}

func registerEndPointToEtcd(ctx context.Context, addr string) {
	// 创建 etcd 客户端
	etcdClient, _ := eclient.NewFromURL(MyEtcdURL)
	etcdManager, _ := endpoints.NewManager(etcdClient, MyService)

	// 创建一个租约，每隔 10s 需要向 etcd 汇报一次心跳，证明当前节点仍然存活
	var ttl int64 = 10
	lease, _ := etcdClient.Grant(ctx, ttl)

	// 添加注册节点到 etcd 中，并且携带上租约 id
	_ = etcdManager.AddEndpoint(
		ctx,
		fmt.Sprintf("%s/%s", MyService, addr),
		endpoints.Endpoint{Addr: addr},
		eclient.WithLease(lease.ID),
	)

	// 每隔 5 s进行一次延续租约的动作
	for {
		select {
		case <-time.After(5 * time.Second):
			// 续约操作
			etcdClient.KeepAliveOnce(ctx, lease.ID)
		case <-ctx.Done():
			return
		}
	}
}

```

## Client

```go
package main

import (
	"context"
	"etcd-demo/pb"
	"fmt"
	clientv3 "go.etcd.io/etcd/client/v3"
	eresolver "go.etcd.io/etcd/client/v3/naming/resolver"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"time"
)

const (
	// grpc 服务名
	MyService = "xiaoxu/demo"
	// etcd 端口
	MyEtcdURL = "http://localhost:2379"
)

func main() {
	etcdClient, err2 := clientv3.NewFromURL(MyEtcdURL)
	if err2 != nil {
		fmt.Println(err2)
		return
	}
	builder, err2 := eresolver.NewBuilder(etcdClient)
	if err2 != nil {
		fmt.Println(err2)
		return
	}
	etcdTarget := fmt.Sprintf("etcd:///%s", MyService)
	dial, err := grpc.Dial(
		etcdTarget,
		grpc.WithResolvers(builder),
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithDefaultServiceConfig(fmt.Sprintf(`{"LoadBalancingPolicy": "%s"}`, "round_robin")),
	)
	if err != nil {
		panic(err)
	}
	client := pb.NewTestDemoClient(dial)
	for true {
		resp, err := client.Demo1(context.Background(), &pb.Req{Msg: "abc"})
		if err != nil {
			panic(err)
		}
		fmt.Println(resp.Msg)
		time.Sleep(time.Second)
	}
}

```

