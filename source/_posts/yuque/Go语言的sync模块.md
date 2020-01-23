
---
title: Go语言的sync模块
date: 2019-09-04 00:11:27 +0800
tags: []
categories: 
---
在程序进行并发操作时，当不同线程使用同一变量时，由于无法判断是否有其他线程在进行写操作，即线程之间出现竞争（资源竞争），因此引入锁机制。<br />Go语言使用sync模块保证线程在操作变量的互斥性。当变量被一个线程操作时，先使用sync将变量上锁，退出操作时，解锁。<br />sync 模块提供基础的同步原语，例如互斥锁（mutual exclusion locks）。除了**Once**和**WaitGroup**类型，大部分的sync模块的方法使用在低级库。更高级别的同步方式是使用通道（channel）和通信（communication）。<br />本文主要介绍sync包内，一些锁的概念及使用方式。
<a name="ELGHO"></a>
## sync包基础
sync包围绕sync.Locker进行，其中interface为：

```go
type Locker interface {
        Lock()
        Unlock()
}
```

<a name="6lkjn"></a>
## 基本锁Mutex
sync.Mutex是互斥锁，相当于在变量入口处确保只有一个线程能够操作变量。<br />假如 num 是一个需要锁处理的存储在共享内存中的变量。通过Mutex处理加锁与解锁。具体例子如下：

```go
import "sync"

type num struct {
	mutex sync.Mutex
    nu int
}

//flash num
func update(num *num) {
    num.mutex.Lock()
    //update num
    num.nu = // new value
    num.mutex.Unlock()  //任何 goroutine 都可用开锁
}
```

<a name="weK7l"></a>
## 单写多读锁RWMutex
顾名思义，允许多个线程进行读取，但仅允许单个线程进行写操作。通过 Rlock() 允许同一时间多个线程读操作；如果使用 Lock() ，则RWMutex变成Mutex。<br />RWMutex的方法有：
```go
    func (rw *RWMutex) Lock()
    func (rw *RWMutex) RLock()
    func (rw *RWMutex) RLocker() Locker
    func (rw *RWMutex) RUnlock()
    func (rw *RWMutex) Unlock()
```
对应的策略有：

- **允许存在多个读锁，但只能有一把写锁**；<br />
- **当写锁未被释放时或此时有正被等待的写锁，读锁不可用**；<br />
- **只有当全部读锁结束，写锁才可用**；<br />

<a name="RSEIF"></a>
## sync.Once
sync.Once可以保证被调用的func，只调用一次。<br />Once数据结构为：

```go
type Once struct {
	m Mutex
	done uint32
}

func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}
	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

由once.Do可以看出，当_once.Do(f)_被调用多次时，只有第一次会调用功能函数f。其核心思想是，使用原子计数记录被执行的次数。其中的锁机制，仍旧使用**Mutex.**

官方给出的一个示例，及对应的执行结果如下：
```go
//官方例子
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}
	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 10; i++ {
		<-done
	}
}

// 打印输出为：
Only once

```

<a name="bJsYV"></a>
## sync.WaitGroup
> A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls Add to set the number of goroutines to wait for. Then each of the goroutines runs and calls Done when finished. At the same time, Wait can be used to block until all goroutines have finished.

WaitGroup可以保证集合内所有goroutines协程完成后，主进程再执行其他动作。避免协程未执行完，主进程就退出或执行其他影响协程的动作。

<a name="aWZUt"></a>
### WaitGroup操作方式
执行协程前使用 func (*WaitGroup) Add 添加等待计数器。协程完成一次，执行func (*WaitGroup) Done，在waitgroup计数器中减少一个，直至计数器为零，释放锁。

<a name="6AL5C"></a>
### 使用举例

```go
package main

import (
	"sync"
)

type httpPkg struct{}

func (httpPkg) Get(url string) {}

var http httpPkg

func main() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
}
```





