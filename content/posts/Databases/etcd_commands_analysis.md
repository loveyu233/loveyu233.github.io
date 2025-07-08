---
title: "etcd分布式锁中使用的etcd命令详解"
date: 2025-06-10T10:11:36+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- etcd
---

# etcd分布式锁中使用的etcd命令详解

## 命令分类概览

分布式锁的实现涉及以下几类etcd操作：
1. **租约管理**: Grant, KeepAlive, Revoke
2. **键值操作**: Put, Get, Delete
3. **事务操作**: Txn (Transaction)
4. **监听操作**: Watch
5. **比较操作**: Compare

## 1. 租约管理命令

### Grant - 创建租约

```go
// 在session.go的NewSession中
resp, err := client.Grant(ops.ctx, int64(ops.ttl))
if err != nil {
    return nil, err
}
id := resp.ID
```

**作用**: 创建一个指定TTL的租约
- **参数**: 
  - `ctx`: 上下文
  - `ttl`: 租约存活时间（秒）
- **返回**: 租约ID
- **用途**: 为分布式锁提供自动过期机制

**etcd命令等价**:
```bash
etcdctl lease grant 60
# 输出: lease 694d71ddacfda227 granted with TTL(60s)
```

### KeepAlive - 租约续约

```go
// 在session.go中
keepAlive, err := client.KeepAlive(ctx, id)
if err != nil || keepAlive == nil {
    cancel()
    return nil, err
}

// 后台goroutine消费续约响应
go func() {
    for range keepAlive {
        // 持续接收keepalive响应
    }
}()
```

**作用**: 持续为租约续期，防止过期
- **参数**: 租约ID
- **返回**: 续约响应通道
- **特点**: 
  - 自动定期发送续约请求
  - 网络断开时停止续约，租约自动过期

**etcd命令等价**:
```bash
etcdctl lease keep-alive 694d71ddacfda227
```

### Revoke - 撤销租约

```go
// 在session.go的Close方法中
ctx, cancel := context.WithTimeout(s.opts.ctx, time.Duration(s.opts.ttl)*time.Second)
_, err := s.client.Revoke(ctx, s.id)
cancel()
```

**作用**: 主动撤销租约，立即释放相关资源
- **参数**: 租约ID
- **用途**: 会话关闭时清理资源

**etcd命令等价**:
```bash
etcdctl lease revoke 694d71ddacfda227
```

## 2. 键值操作命令

### Put - 写入键值

```go
// 在tryAcquire方法中
put := v3.OpPut(m.myKey, "", v3.WithLease(s.Lease()))
```

**作用**: 创建或更新键值对
- **参数**:
  - `key`: 键名
  - `value`: 键值（锁场景中通常为空字符串）
  - `v3.WithLease()`: 绑定到租约
- **选项**:
  - `WithLease(leaseID)`: 将键绑定到指定租约

**etcd命令等价**:
```bash
etcdctl put /my-lock/1a2b3c4d5e6f "" --lease=694d71ddacfda227
```

### Get - 读取键值

#### 普通Get

```go
// 在Lock方法中验证会话
gresp, werr := client.Get(ctx, m.myKey)
if len(gresp.Kvs) == 0 {
    return ErrSessionExpired
}
```

**作用**: 获取指定键的值和元数据
- **返回**: 键值对数组和元数据

**etcd命令等价**:
```bash
etcdctl get /my-lock/1a2b3c4d5e6f
```

#### 带选项的Get

```go
// 在tryAcquire中获取锁持有者
getOwner := v3.OpGet(m.pfx, v3.WithFirstCreate()...)
```

**重要选项**:
- `v3.WithFirstCreate()`: 获取CreateRevision最小的键
- `v3.WithPrefix()`: 前缀匹配获取多个键
- `v3.WithCountOnly()`: 只返回数量
- `v3.WithLimit(n)`: 限制返回数量

**etcd命令等价**:
```bash
# 获取前缀下所有键，按CreateRevision排序，取第一个
etcdctl get /my-lock/ --prefix --sort-by=CREATE --limit=1
```

### Delete - 删除键

```go
// 在Unlock方法中
if _, err := client.Delete(ctx, m.myKey); err != nil {
    return err
}

// 在TryLock失败时清理
if _, err := client.Delete(ctx, m.myKey); err != nil {
    return err
}
```

**作用**: 删除指定的键
- **参数**: 键名
- **用途**: 释放锁，通知等待者

**etcd命令等价**:
```bash
etcdctl del /my-lock/1a2b3c4d5e6f
```

## 3. 事务操作命令

### Txn - 事务

```go
// tryAcquire中的完整事务
resp, err := client.Txn(ctx).
    If(cmp).                    // 条件
    Then(put, getOwner).        // 成功时执行
    Else(get, getOwner).        // 失败时执行
    Commit()                    // 提交事务
```

**事务结构**:
```
IF condition
THEN operation1, operation2, ...
ELSE operation3, operation4, ...
```

**特点**:
- **原子性**: 所有操作要么全部成功，要么全部失败
- **条件执行**: 基于条件选择不同的操作分支
- **批量操作**: 一次RPC执行多个操作

**etcd命令等价**:
```bash
etcdctl txn <<EOF
compares:
create_revision("/my-lock/1a2b3c4d5e6f") = "0"

success_requests:
put /my-lock/1a2b3c4d5e6f "" --lease=694d71ddacfda227
get /my-lock/ --prefix --sort-by=CREATE --limit=1

failure_requests:
get /my-lock/1a2b3c4d5e6f
get /my-lock/ --prefix --sort-by=CREATE --limit=1
EOF
```

## 4. 比较操作

### Compare - 条件比较

```go
// 检查键是否不存在
cmp := v3.Compare(v3.CreateRevision(m.myKey), "=", 0)

// 检查是否为锁持有者（在IsOwner方法中）
func (m *Mutex) IsOwner() v3.Cmp {
    return v3.Compare(v3.CreateRevision(m.myKey), "=", m.myRev)
}
```

**比较类型**:
- `v3.CreateRevision(key)`: 键的创建版本号
- `v3.ModRevision(key)`: 键的修改版本号  
- `v3.Version(key)`: 键的版本号
- `v3.Value(key)`: 键的值
- `v3.Lease(key)`: 键的租约ID

**比较操作符**:
- `"="`: 等于
- `"!="`: 不等于
- `">"`: 大于
- `"<"`: 小于

## 5. 监听操作命令

### Watch - 监听变化

```go
// waitDeletes函数中（未在提供代码中显示，但被调用）
wc := client.Watch(ctx, pfx, v3.WithPrefix(), v3.WithRev(maxRev+1))
for wresp := range wc {
    for _, ev := range wresp.Events {
        if ev.Type == etcdserverpb.DELETE {
            // 处理删除事件
            return checkTurn(ctx, client, pfx, myRev)
        }
    }
}
```

**作用**: 监听键的变化事件
- **事件类型**:
  - `PUT`: 键被创建或更新
  - `DELETE`: 键被删除
- **选项**:
  - `v3.WithPrefix()`: 监听前缀匹配的所有键
  - `v3.WithRev(rev)`: 从指定版本开始监听
  - `v3.WithPrevKV()`: 返回变化前的键值

**etcd命令等价**:
```bash
# 监听前缀下的所有变化
etcdctl watch /my-lock/ --prefix

# 从指定版本开始监听
etcdctl watch /my-lock/ --prefix --rev=100
```

## 6. 操作组合示例

### 完整的锁获取流程涉及的命令序列

```go
// 1. 创建会话 - Grant + KeepAlive
session, _ := concurrency.NewSession(client)

// 2. 尝试获取锁 - Txn(Compare + Put + Get)
mutex := concurrency.NewMutex(session, "/my-lock")
err := mutex.Lock(ctx)

// 3. 如果需要等待 - Watch
// 监听前面锁的释放

// 4. 使用完毕释放锁 - Delete
err = mutex.Unlock(ctx)

// 5. 关闭会话 - Revoke
session.Close()
```

## 7. 关键设计模式

### 基于版本号的排序

```go
// 利用CreateRevision实现公平锁
// CreateRevision小的键先获得锁
ownerKey := resp.Responses[1].GetResponseRange().Kvs
if len(ownerKey) == 0 || ownerKey[0].CreateRevision == m.myRev {
    // 获得锁
}
```

### 租约绑定模式

```go
// 所有锁键都绑定到会话租约
put := v3.OpPut(m.myKey, "", v3.WithLease(s.Lease()))
```

**优势**:
- 客户端崩溃时，租约自动过期
- 绑定的键自动删除，释放锁
- 避免死锁情况

### 事务优化模式

```go
// 一次RPC完成多个操作
client.Txn(ctx).
    If(condition).
    Then(op1, op2).
    Else(op3, op4).
    Commit()
```

**优势**:
- 减少网络往返
- 保证操作原子性
- 提高性能

## 8. 命令使用统计

| 命令类型 | 使用频率 | 关键用途 |
|---------|---------|----------|
| Grant | 一次/会话 | 创建租约 |
| KeepAlive | 持续 | 维持会话 |
| Txn | 每次获锁 | 原子操作 |
| Get | 多次 | 状态查询 |
| Put | 每次获锁 | 创建锁键 |
| Delete | 每次解锁 | 释放锁 |
| Watch | 等待时 | 监听变化 |
| Compare | 事务中 | 条件判断 |
| Revoke | 会话结束 | 清理资源 |

## 总结

etcd分布式锁通过巧妙组合这些基础命令，实现了：

1. **可靠性**: 通过租约机制防止死锁
2. **公平性**: 通过版本号排序实现公平获取
3. **高性能**: 通过事务减少RPC次数
4. **实时性**: 通过Watch机制及时响应变化

这种设计充分利用了etcd的MVCC、租约、事务和监听等核心特性，是分布式协调服务的经典应用案例。