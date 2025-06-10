---
title: "pprof"
date: 2023-07-15T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- pprof
- go test
---

# go test [包路径] [选项]

>- `-v`：输出详细的测试日志，包括每个测试函数的名称和执行结果。
>- `-run <正则表达式>`：指定要运行的测试函数或测试组的正则表达式。
>- `-cover`：计算测试覆盖率，并显示每个文件的覆盖率百分比。
>- `-bench <正则表达式>`：运行性能基准测试，并显示基准测试的结果。
>- `-timeout <时长>`：设置超时时间，单个测试的最大执行时间。
>- `-count <次数>`：执行测试的次数，默认为 1 次。
>- -benchmem
>  打印出申请内存的次数。一般用于简单的性能测试，不会导出数据文件。
>- -blockprofile block.out
>  将协程的阻塞数据写入特定的文件（block.out）。如果-c，则写成二进制文件。
>- -cpuprofile cpu.out
>  将协程的CPU使用数据写入特定的文件（cpu.out）。如果-c，则写成二进制文件。
>- -memprofile mem.out
>  将协程的内存申请数据写入特定的文件（mem.out）。如果-c，则写成二进制文件。
>- -mutexprofile mutex.out
>  将协程的互斥数据写入特定的文件（mutex.out）。如果-c，则写成二进制文件。
>- -trace trace.out
>  将执行调用链写入特定文件（trace.out）。

例如，要运行当前目录下所有测试文件的测试，可以执行以下命令：

```
bashCopy Codego test
```

如果你只想运行某个特定包的测试，可以指定包的导入路径：

```
bashCopy Codego test github.com/example/package
```

你还可以使用 `-run` 标志来选择性地运行特定的测试函数或测试组。例如，下面的命令将运行名称中包含 "TestFoo" 的测试函数：

```
bashCopy Codego test -run TestFoo
```

如果你想在测试运行时生成并查看测试覆盖率报告，可以使用 `-cover` 标志：

```bash
bashCopy Codego test -cover
```





# 项目配置

1. 在项目中导入

```go
_ "net/http/pprof"
```

2. 启动http服务

```go
http.ListenAndServe(":6060", nil)
```

3. 访问网址

```url
http://27.0.0.1:6060/debug/pprof
```

>- allocs：查看过去所有内存分配的样本（历史累计）。
>- block：查看导致阻塞同步的堆栈跟踪（历史累计）。
>- cmdline： 当前程序的命令行的完整调用路径（从程序一开始运行时决定）。
>- goroutine：查看当前所有运行的 goroutines 堆栈跟踪（实时变化）。
>- heap：查看活动对象的内存分配情况（实时变化）。
>- mutex：查看导致互斥锁的竞争持有者的堆栈跟踪（历史累计）。
>- profile： 默认进行 30s 的 CPU Profiling，得到一个分析用的 profile 文件（从开始分析，到分析结束）。
>- threadcreate：查看创建新 OS 线程的堆栈跟踪（一般没啥用）。

> 默认情况下是不追踪block和mutex的信息,开启方式:

```go
runtime.SetBlockProfileRate(1) // 开启对阻塞操作的跟踪，block  
runtime.SetMutexProfileFraction(1) // 开启对锁调用的跟踪，mutex
```



# 可视化安装

>安装并配置graphviz/bin到环境变量中

```url
https://graphviz.org/download/
```



# go tool pprof



## 直接获取并分析

> go tool pprof http://localhost:6060/debug/pprof/XXX
>
> 自动下载到本地并分析,执行后会提示分析文件的保存位置
>
> Saved profile in C:\Users\user\pprof\pprof.___go_build_goTest.exe.contentions.delay.002.pb.gz



## 下载到本地再分析

```shell
curl -o 目录名称 http://localhost:6060/debug/pprof/XXX
go tool pprof 目录名称
```



## go test

>```bash
>-bench=.
>进行性能测试，“.”是正则匹配，匹配了所有的测试函数
>
>-benchmem
>打印出申请内存的次数。一般用于简单的性能测试，不会导出数据文件。
>
>-blockprofile block.out
>将协程的阻塞数据写入特定的文件（block.out）。如果-c，则写成二进制文件。
>
>-cpuprofile cpu.out
>将协程的CPU使用数据写入特定的文件（cpu.out）。如果-c，则写成二进制文件。
>
>-memprofile mem.out
>将协程的内存申请数据写入特定的文件（mem.out）。如果-c，则写成二进制文件。
>
>-mutexprofile mutex.out
>将协程的互斥数据写入特定的文件（mutex.out）。如果-c，则写成二进制文件。
>
>-trace trace.out
>将执行调用链写入特定文件（trace.out）。
>```



## 代码获取文件

>其他的，包括alloc、heap、mutex、block、goroutine、threadcreate，写法上都是一样的，只需要替换那个字符串就行了。

```go
f, _ := os.Create("cpu.pprof")
defer f.Close()
// cpu文件的获取
_ = pprof.StartCPUProfile(f)
time.Sleep(1 * time.Minute)
pprof.StopCPUProfile()

// 其他
f, _ := os.Create("XXX.pprof")
defer f.Close()
_ = pprof.Lookup("allocs").WriteTo(f, 0) // 前面这个字符串
```



# 获取测试信息

>在执行完 go tool pprof xxx 后会等待新的输入
>
>top: 文本分析
>
>web: 开启一个http服务可视化的查看



## 更方便的方式

```shell
go tool pprof -http=:端口 分析文件位置
分析文件位置可以是: 目录, pprof.XXX.samples.cpu.001.pb.gz http://127.0.0.1:6060/debug/pprof/xxx
```



# 指标理解



## flat flat%

>一个函数内``直接操作``的物理耗时,a函数调用b函数,则调用的b函数的执行时间不算到flat
>
>所有的flat相加即是总采样时间，所有的flat%相加应该等于100%。



## cum cum%

>一个函数内的全部操作耗时, 包括函数内调用的其他函数
>
>cum%即是cum的时间/总运行时间。内存等参数同理。
>
>cum%其上所有行的flat%的累加。可以视为，这一行及其以上行，其所有的directly操作一共占了多少物理时间。



sum%

>函数自身累积使用 CPU 总比例



Name

>函数名



## 连线图

格式: 

>包名
>
>函数名
>
>flat(flat%)
>
>cum(cum%)



>节点的颜色越红，其cum和cum%越大。其颜色越灰白，则cum和cum%越小。
>
>节点越大，其flat和flat%越大；其越小，则flat和flat%越小
>
>线条代表了函数的调用链，线条越粗，代表指向的函数消耗了越多的资源。反之亦然。
>
>线条的样式代表了调用关系。实线代表直接调用；虚线代表中间少了几个节点；带有inline字段表示该函数被内联进了调用方（不用在意，可以理解成实线）。



## 火焰图

>火焰图的横向长度表示cum，相比下面超出的一截代表flat。



# trace

>go tool trace 文件