---
title: "ants的CAS简易实现"
date: 2023-10-25T11:23:35+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- cas
- ants
---

>通过对``spinLock``的``atomic``原子操作来达到加解锁目的

```go
type spinLock uint32

const maxBackoff = 16

func (s *spinLock) Lock() {
	backoff := 1
	for !atomic.CompareAndSwapUint32((*uint32)(s), 0, 1) {
		for i := 0; i < backoff; i++ {
			runtime.Gosched()
		}
		if backoff < maxBackoff {
			backoff <<= 1
		}
	}
}
func (s *spinLock) UnLock() {
	atomic.StoreUint32((*uint32)(s), 0)
}
```

