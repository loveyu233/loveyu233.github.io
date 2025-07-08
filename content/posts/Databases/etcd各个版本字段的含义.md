---
date: '2025-06-11T10:08:27+08:00'
title: 'Etcd各个版本字段的含义'
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- etcd
---

## 1. Version（版本）

这是键值对的版本号，表示该键被修改的次数。每次对某个键进行写操作（PUT/DELETE）时，version 都会递增 1。这个版本号是针对单个键的，不同的键有各自独立的 version。

## 2. Revision（修订版本）

这是 etcd 集群的全局修订版本号，是一个单调递增的数字。每当集群中发生任何写操作时，revision 都会增加。它代表了整个集群状态的版本，可以用来实现一致性读取和 MVCC（多版本并发控制）。

## 3. ModRevision（修改修订版本）

这表示某个键最后一次被修改时的 revision 值。当你创建或更新一个键时，该键的 mod_revision 会被设置为当前的集群 revision。

## 4. CreateRevision（创建修订版本）

这表示某个键被创建时的 revision 值。一旦键被创建，这个值就不会改变，除非键被删除后重新创建。

## 代码

```go
func TestVersion(t *testing.T) {
	ctx := context.Background()

	resp1, _ := etcdClient.Put(ctx, "test-key", "value1")
	fmt.Printf("第一次写入后 Revision: %d\n", resp1.Header.Revision)

	// 获取键的详细信息
	getResp, _ := etcdClient.Get(ctx, "test-key")
	if len(getResp.Kvs) > 0 {
		kv := getResp.Kvs[0]
		fmt.Printf("Key: %s\n", kv.Key)
		fmt.Printf("Value: %s\n", kv.Value)
		fmt.Printf("Version: %d\n", kv.Version)               // 键的版本号
		fmt.Printf("CreateRevision: %d\n", kv.CreateRevision) // 创建时的集群版本
		fmt.Printf("ModRevision: %d\n", kv.ModRevision)       // 最后修改时的集群版本
	}

	// 第二次更新同一个键
	resp2, _ := etcdClient.Put(ctx, "test-key", "value2")
	fmt.Printf("第二次写入后 Revision: %d\n", resp2.Header.Revision)

	// 再次获取键信息
	getResp2, _ := etcdClient.Get(ctx, "test-key")
	if len(getResp2.Kvs) > 0 {
		kv := getResp2.Kvs[0]
		fmt.Printf("更新后 Version: %d\n", kv.Version)               // 版本号增加了
		fmt.Printf("更新后 CreateRevision: %d\n", kv.CreateRevision) // 创建版本不变
		fmt.Printf("更新后 ModRevision: %d\n", kv.ModRevision)       // 修改版本更新了
	}

	// 第二次更新同一个键
	etcdClient.Delete(ctx, "test-key")
	fmt.Printf("第二次写入后 Revision: %d\n", resp2.Header.Revision)

	resp5, _ := etcdClient.Put(ctx, "test-key", "value2")
	fmt.Printf("删除后写入 Revision: %d\n", resp5.Header.Revision)
	// 再次获取键信息
	getResp4, _ := etcdClient.Get(ctx, "test-key")
	if len(getResp4.Kvs) > 0 {
		kv := getResp4.Kvs[0]
		fmt.Printf("删除后写入 Version: %d\n", kv.Version)               // 版本号增加了
		fmt.Printf("删除后写入 CreateRevision: %d\n", kv.CreateRevision) // 创建版本不变
		fmt.Printf("删除后写入 ModRevision: %d\n", kv.ModRevision)       // 修改版本更新了
	}
}

```

### 输出

```shell
第一次写入后 Revision: 606
Key: test-key
Value: value1
Version: 1
CreateRevision: 606
ModRevision: 606
第二次写入后 Revision: 607
更新后 Version: 2
更新后 CreateRevision: 606
更新后 ModRevision: 607
第二次写入后 Revision: 607
删除后写入 Revision: 609
删除后写入 Version: 1
删除后写入 CreateRevision: 609
删除后写入 ModRevision: 609
```

