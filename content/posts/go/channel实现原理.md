---
title: "Channel实现原理"
date: 2023-10-28T16:06:27+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- channel
- 源码
---

# 数据结构

![数据结构](https://www.loveyu.asia//img/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)

```go
type hchan struct {
	qcount   uint           // 当前 channel 中存在多少个元素
	dataqsiz uint           // 当前 channel 能存放的元素容量
	buf      unsafe.Pointer // channel 中用于存放元素的环形缓冲区
	elemsize uint16         // chan元素类型大小
	closed   uint32         // chan是否关闭
	elemtype *_type         // chan元素的类型
	sendx    uint           // 发送元素进入环形缓冲区的 index
	recvx    uint           // 接收元素所处的环形缓冲区的 index
	recvq    waitq          // 因接收而陷入阻塞的协程队列
	sendq    waitq          // 因发送而陷入阻塞的协程队列
	lock     mutex          // 锁
}
type waitq struct {
	first *sudog // 队头
	last  *sudog // 队尾
}

type sudog struct {
	g *g // goroutine

	next *sudog // 下一个节点指针
	prev *sudog // 上一个节点指针
  
  elem unsafe.Pointer // 读取/写入 channel 的数据的容器
  
  isSelect bool  // 标识当前协程是否处在 select 多路复用的流程中
  
  c        *hchan // 标识与当前 sudog 交互的 chan
}

```

# 初始化

```go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	// 判断元素大小是否超出限制
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
  // 判断对齐方式是否正确
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}

  // 计算需要分配的内存大小，并进行溢出和范围检查。如果溢出或者大小超过了允许的最大值抛出异常
	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	var c *hchan
	switch {
  // 无缓冲
	case mem == 0:
  	// 分配hchanSize大小的内存并转为*hchan类型
    // 则仅申请一个大小为默认值 96 的空间
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		c.buf = c.raceaddr()
  // 有缓冲的 struct 型
	case elem.ptrdata == 0:
    //一次性分配好 96 + mem 大小的空间，并且调整 chan 的 buf 指向 mem 的起始位置
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
  // 指针类型元素
	default:
    // 分别申请 chan 和 buf 的空间，因为是指针类型的所以两者无需连续
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

  // 初始化元素类型大小
	c.elemsize = uint16(elem.size)
  // 元素类型
	c.elemtype = elem
  // 容量
	c.dataqsiz = uint(size)
	// 锁
  lockInit(&c.lock, lockRankHchan)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```

# 发送

![chansend](https://www.loveyu.asia//img/chansend.png)

>1. 对没有初始化的channel进行发送数据会死锁
>2. 对关闭的channel进行发送数据会报错
>3. 发送数据时如果该管道对应的读操作有阻塞的goroutine则直接把数据拷贝到被阻塞的读goroutine
>4. 发送数据时如果该管道内还有容量则把数据插入对应的槽位
>5. 发送数据时如果该管道内没有容量则把当前goroutine封装后放入到阻塞写队列中然后挂起等待唤醒

```go
// block 用来标识是否为阻塞,在select中使用则为不阻塞
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	// 如果chan没有初始化则死锁
  if c == nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}
	
  // 加锁
	lock(&c.lock)

  // 如果chan被关闭则报错
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

  // 写数据时存在阻塞读协程,从读堵塞队列中拿到第一个被阻塞的goroutine封装对象sudog
	if sg := c.recvq.dequeue(); sg != nil {
    // 调用send方法直接把写数据拷贝给该读goroutine并在拷贝后执行解锁操作
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}

  // 写数据时无阻塞读协程但环形缓冲区仍有空间
	if c.qcount < c.dataqsiz {
    // 获取环形缓冲数组中发送索引的槽位指针
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			racenotify(c, c.sendx, nil)
		}
    // 把数据写入到该槽位
		typedmemmove(c.elemtype, qp, ep)
    // 发送索引++
		c.sendx++
    // 如果索引值等于数组容量值则赋值为0(以此来实现环形数组)
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
    // 环形数组的当前存在元素数量++
		c.qcount++
    // 解锁
		unlock(&c.lock)
		return true
	}
	
  // 如果为非阻塞则直接解锁返回
	if !block {
		unlock(&c.lock)
		return false
	}

	// 获取到当前的goroutine
	gp := getg()
  // 构造封装当前 goroutine 的 sudog 对象
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
  // 把 sudog 添加到当前 channel 的阻塞写协程队列中
	c.sendq.enqueue(mysg)
	gp.parkingOnChan.Store(true)
  // gopark 挂起当前协程
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// KeepAlive将其参数标记为当前可访问。这确保了在程序中调用KeepAlive的点之前，对象不会被释放
	KeepAlive(ep)

	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
  // 倘若协程从 park 中被唤醒，则回收 sudog（sudog能被唤醒，其对应的元素必然已经被读协程取走）
	gp.waiting = nil
	gp.activeStackChans = false
	closed := !mysg.success
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	if closed {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	return true
}
```



# 读取

![recv](https://www.loveyu.asia//img/recv.png)

>1. 读没有初始化的channel死锁
>2. 读已经关闭的channel且channel内数据则返回false
>3. 读时有写阻塞的队列
>   1. 无缓冲则直接读取写goroutine数据然后唤醒写goroutine
>   2. 有缓存则读取缓冲区头部数据然后唤醒写goroutine
>4. 读时无写阻塞队列
>   1. 缓冲区有数据直接读取返回
>   2. 缓冲区无数据挂起阻塞

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	if debugChan {
		print("chanrecv: chan=", c, "\n")
	}

  // channel没有初始化则死锁
	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	// 如果为非阻塞且队列为空则直接返回
	if !block && empty(c) {
		if atomic.Load(&c.closed) == 0 {
			return
		}
		if empty(c) {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)
	// 如果读已经关闭的channel
	if c.closed != 0 {
    // 且channel内无数据
		if c.qcount == 0 {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
      // 解锁并返回false标志没有读取到数据
			unlock(&c.lock)
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
		// 如果通道已关闭但缓冲区有数据，则继续执行后边代码
    
  // 没有关闭
	} else {
    // 如果 channel 未关闭，则尝试从等待的发送者队列中获取数据
		if sg := c.sendq.dequeue(); sg != nil {
			recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
			return true, true
		}
	}

  // 如果 channel 中有数据，则直接从 channel 中获取数据
	if c.qcount > 0 {
		// Receive directly from queue
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			racenotify(c, c.recvx, nil)
		}
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

  // 非阻塞直接返回
	if !block {
		unlock(&c.lock)
		return false, false
	}

  // 获取当前goroutine添加到读阻塞队列中,然后调用gopark挂起
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	gp.parkingOnChan.Store(true)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// 当可以添加数据时唤醒
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	success := mysg.success
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, success
}
```

# 关闭

![close](https://www.loveyu.asia//img/close.png)

>- 关闭未初始化过的 channel 会 panic；
>- 加锁；
>- 重复关闭 channel 会 panic；
>- 将阻塞读协程队列中的协程节点统一添加到 glist；
>- 将阻塞写协程队列中的协程节点统一添加到 glist；
>- 唤醒 glist 当中的所有协程.

```go
func closechan(c *hchan) {
  	// 关闭未初始化的channel直接报错
    if c == nil {
        panic(plainError("close of nil channel"))
    }

    lock(&c.lock)
    // 重复关闭channel直接报错
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("close of closed channel"))
    }

    c.closed = 1

    var glist gList
    // 遍历该channel的读阻塞队列到glist
    for {
        sg := c.recvq.dequeue()
        if sg == nil {
            break
        }
        if sg.elem != nil {
            typedmemclr(c.elemtype, sg.elem)
            sg.elem = nil
        }
        gp := sg.g
        gp.param = unsafe.Pointer(sg)
        sg.success = false
        glist.push(gp)
    }

    // // 遍历该channel的写阻塞队列到glist
    for {
        sg := c.sendq.dequeue()
        if sg == nil {
            break
        }
        sg.elem = nil
        gp := sg.g
        gp.param = unsafe.Pointer(sg)
        sg.success = false
        glist.push(gp)
    }
    unlock(&c.lock)

    // 唤醒全部阻塞中的goroutine
    for !glist.empty() {
        gp := glist.pop()
        gp.schedlink = 0
        goready(gp, 3)
      
}
```

