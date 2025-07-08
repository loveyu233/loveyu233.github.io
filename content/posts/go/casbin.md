---
title: "go使用casbin"
date: 2020-06-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- casbin
---

官网：

>https://casbin.org/zh-CN/

`sub, obj, act` 表示经典三元组: 访问实体 (Subject)，访问资源 (Object) 和访问方法 (Action)，还有一个eft这条属性默认就是allow，不声明就是默认的，如果声明的话改属性只能是allow/deny

## ACL模型

例：

<img src="../../../img/acl.png" alt="acl" style="zoom:100%;" />

>[request_definition]
>r = sub, obj, act
>
>对应Request的 alice, data1, read  也就是sub=alice；obj=data1；act=read
>
>[policy_definition]
>p = sub, obj, act
>
>对应policy的 p, alice, data1, read ；p, bob, data2, write  属性
>
>[matchers]
>m = r.sub   p.sub && r.obj   p.obj && r.act   p.act
>
>R和P的对应法则
>
>[policy_effect]
>e = some(where (p.eft   allow))
>
>只能是以下几条不能自定义
>
>some(where (p.eft   allow))；
>
>!some(where (p.eft   deny))；
>
>some(where (p.eft   allow)) && !some(where (p.eft   deny))；
>
>priority(p.eft) || deny；subjectPriority(p.eft) 

## RBAC模型

<img src="../../../img/rbac.png" alt="acl" style="zoom:100%;" />

>该模型多出一个属性 [role_definition]   g = _, _ 
>
>可以理解为第一个 下划线 指的是角色；第二个 下划线 指的是组
>
>RBAC模型的匹配理解为：
>
>先拿 r的sub 值到 Policy的g 中查找和第一个 下划线 一样的，查找到后再拿对应的 g的第二个下划线 值和 r的obj和act 两个值共三个值和 p的三个值比对  如果相同就通过
>
>这里会有两种情况， 如果上边情况不通过但是r的三条属性有对应的p的三条属性一样的就会通过  
>
>可以理解为：
>
>1. 这个组不允许但是这个组的这个用户却允许，这样的情况一样会返回true也就是通过
>
>2. 如果这个用户不允许但是这个组允许，这样的情况也会返回true
>
>正常情况下希望的是用第二种情况来通过，因为第一种情况的通过就没必要用RBAC模型了，ACl就是第一种情况的

## 域内RBAC模型

<img src="../../../img/yrbac.png" alt="acl" style="zoom:100%;" />

>这个模型属于RBAC的另一种情况
>
>区别就是 g多了一个下划线 ； 其他的r和p也多了一个属性dom   
>
>g的这个下划线指的就是 域 
>
>理解为：
>
>用户，用户组，用户组的域
>
>这时的判别方式就是：
>
>用r的sub和dom到policy中查找对应g的第一个下划线和第三个下划线  没有直接返回false， 有的话再拿对应g的第二个下划线值和sub，dom，obj以及act这四条属性到policy中查找是否有对应一样的，有就返回true  
>
>这里返回true的情况也是两种：
>
>1. r的四条值再polcy种有对应一样的，这时就算没有对应g也一样返回true
>2. r的前两个值比对g的第一个和第三个值，拿到对比成功的g的第二个值和第三个值以及r的后两个值的这四个值去对比policy的p对应四个值一样返回true



<img src="../../../img/rbacPattern.png" alt="acl" style="zoom:100%;" />



> 和RBAC一样



## 代码使用：

### 第一：不用数据库

```go
package main

import (
   "fmt"
   "github.com/casbin/casbin/v2"
)

func main() {
   e, err := casbin.NewEnforcer("demo1/model.conf", "demo1/policy.csv")
   if err != nil {
      fmt.Println(err)
   }
   sub := "alice" // 想要访问资源的用户。
   obj := "data1" // 将被访问的资源。
   act := "read"  // 用户对资源执行的操作。

   //添加
   policy, err := e.AddPolicy("alice", "data1", "read")
   if err != nil {
      fmt.Println(err)
   }
   //添加是否成功
   if policy {
      fmt.Println("add success")
   }

   //查找对应的p
   ok, err := e.Enforce(sub, obj, act)

   if err != nil {
      // 处理err
      fmt.Println(err)
   }

   if ok   true {
      // 允许alice读取data1
      fmt.Println("success")
   } else {
      // 拒绝请求，抛出异常
      fmt.Println("no success")
   }
}
```

model.conf

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft   allow))

[matchers]
m = r.sub   p.sub && r.obj   p.obj && r.act   p.act
```

policy.csv

```
p,zhangsan,data1,read
```

### 第二：使用数据库

使用数据库就是吧policy.CSV的属性添加到数据库中保存，这里的数据库操作使用gorm

```go
package main

import (
   "fmt"
   "github.com/casbin/casbin/v2"
   _ "github.com/casbin/casbin/v2"
   gormadapter "github.com/casbin/gorm-adapter/v3"
   _ "github.com/go-sql-driver/mysql"
)

func judgePolicy(e *casbin.Enforcer, sub, obj, act string) {
   ok, err := e.Enforce(sub, obj, act)
   if err != nil {
      // 处理err
      fmt.Println(err)
   }
   if ok   true {
      // 允许alice读取data1
      fmt.Println("true")
   } else {
      // 拒绝请求，抛出异常
      fmt.Println("false")
   }
}
func addPolicy(e *casbin.Enforcer, sub, obj, act string) {
   policy, err := e.AddPolicy(sub, obj, act)
   if err != nil {
      fmt.Println(err)
   }
   if policy {
      fmt.Println("添加成功")
   }
}
func selectPolicy(e *casbin.Enforcer, i int, s string) {
   policy := e.GetFilteredPolicy(i, s)
   for i2 := range policy {
      fmt.Println(policy[i2])
   }
}
func removePolicy(e *casbin.Enforcer, sub, obj, act string) {
   policy, err := e.RemovePolicy(sub, obj, act)
   if err != nil || !policy {
      fmt.Println("删除失败")
   } else {
      fmt.Println("删除成功")
   }
}
func updatePolicy(e *casbin.Enforcer, o, n []string) {
   policy, err := e.UpdatePolicy(o, n)
   if err != nil || !policy {
      fmt.Println("修改失败")
   } else {
      fmt.Println("修改成功")
   }
}

func addGroupPolicy(e *casbin.Enforcer, s1, s2 string) {
   policy, err := e.AddGroupingPolicy(s1, s2)
   if err != nil || !policy {
      fmt.Println("添加group失败")
   } else {
      fmt.Println("添加group成功")
   }
}
func main() {
   a, _ := gormadapter.NewAdapter("mysql", "root:mysql233..@tcp(127.0.0.1:3306)/casbin?charset=utf8&parseTime=True&loc=Local", true) // Your driver and data source.
   e, _ := casbin.NewEnforcer("demo2/model.conf", a)
   sub := "guest" // 想要访问资源的用户。
   obj := "/v1"   // 将被访问的资源。
   act := "GET"   // 用户对资源执行的操作。

   //addPolicy(e, sub, obj, act)
   //selectPolicy(e, 2, "read")
   //removePolicy(e, sub, obj, act)
   //updatePolicy(e, []string{sub, obj, act}, []string{"a", "b", "c"})

   //addGroupPolicy(e, "alice", "g1")
   judgePolicy(e, sub, obj, act)
}
```
