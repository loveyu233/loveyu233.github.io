---
title: "实现支持高并发且支持堵塞和超时控制的map"
date: 2023-10-26T19:43:57+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- map
- chan
---



# Map

>1. 高并发
>2. 只有插入和查询操作
>3. 查询时key存在则返回value,不存在则阻塞等待k v对存入,可以指定超时时间,超过则返回超时报错

第一种:

```go

type MyConcurrentMap struct {
	cMap map[int]int
	sync.Locker
	KChan map[int]chan struct{}
}

func (m *MyConcurrentMap) Set(k, v int) {
	m.Lock()
	defer m.Unlock()
	m.cMap[k] = v
	ch, ok := m.KChan[k]
	if !ok {
		return
	}
	select {
  // 如果读取到数据说明该管道已经被关闭了,就不能重复关闭
	case <-ch:
		return
	default:
    // 关闭管道达到通知所有阻塞管道的目的
		close(ch)
	}
}

func (m *MyConcurrentMap) Get(k int, maxWaitingDuration time.Duration) (int, error) {
	m.Lock()
	v, ok := m.cMap[k]
	if ok {
		m.Unlock()
		return v, nil
	}
	vc, ok := m.KChan[k]
	if !ok {
		vc = make(chan struct{})
		m.KChan[k] = vc
	}
	ctx, cancelFunc := context.WithTimeout(context.Background(), maxWaitingDuration)
	defer cancelFunc()
	m.Unlock()
	select {
	case <-ctx.Done():
		return -1, ctx.Err()
	case <-vc:
	}
	m.Lock()
	v = m.cMap[k]
	m.Unlock()
	return v, nil
}

```

第二种

```go
type MyChan struct {
	ch chan struct{}
	sync.Once
}

func (m *MyChan) Close() {
  // 达到只会执行一次的目的
	m.Do(func() {
		close(m.ch)
	})
}

func NewMyChan() *MyChan {
	return &MyChan{
		ch: make(chan struct{}),
	}
}

type MyConcurrentMap struct {
	cMap map[int]int
	sync.Locker
	KChan map[int]*MyChan
}

func (m *MyConcurrentMap) Set(k, v int) {
	m.Lock()
	defer m.Unlock()
	m.cMap[k] = v
	ch, ok := m.KChan[k]
	if !ok {
		return
	}
  // 直接关闭,因为Close是使用once的,所以无论调用多少次但只会关闭一次管道
	ch.Close()
}

func (m *MyConcurrentMap) Get(k int, maxWaitingDuration time.Duration) (int, error) {
	m.Lock()
	v, ok := m.cMap[k]
	if ok {
		m.Unlock()
		return v, nil
	}
	vc, ok := m.KChan[k]
	if !ok {
		vc = NewMyChan()
		m.KChan[k] = vc
	}
	ctx, cancelFunc := context.WithTimeout(context.Background(), maxWaitingDuration)
	defer cancelFunc()
	m.Unlock()
	select {
	case <-ctx.Done():
		return -1, ctx.Err()
	case <-vc.ch:
	}
	m.Lock()
	v = m.cMap[k]
	m.Unlock()
	return v, nil
}
```

