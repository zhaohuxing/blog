---
itle: "Go 理论 - 笔记"
date: 2021-05-20T00:00:00+08:00
draft: false
tags: ["go"]
---

## Go 内存分配

内存空间包含两个重要的区域：栈和堆。Go 语言的内存分配由标准库 runtime.newobject 自动完成的。会根据分配内存大小分为两种策略，以 32KB 为界限。

对于大于 32KB 直接从堆上分配。

对于小于等于 32KB 的，Go 语言会从 mcache 本地缓存中为其分配内存。该缓存处理内存块大小为 32KB 的可分配内存链表，名为 mspn。Groutinue 会使用当前处理器 P 来透过 mcache 找到 span 列表中第一个可用的 object。

span 的列表按所存储的 object 分为了 67 个等级，其容量从 8 字节到 32KB。数量从 1024 到 1。而 span 的大小不是固定的，是 8 KB 到 32 KB 的连续内存。

如果本地 mcache 上的 span 用尽了，它会从 mcentral 上获取一个新的 span 链表，加到本地链表中。mcentral 维护双向链表的 span，管理 span 链表的。 

如果 mcentral 中的 span 也用尽了，Go 语音透过 OS 从 heap 上申请新的内存加到 mcentral 链表中。 

## Go 垃圾回收

#### 标记-清除

垃圾收集器从垃圾收集的根对象出发，递归遍历这些对象指向的子对象并将所有可达的对象标记成存活；标记阶段结束后，垃圾收集器会依次遍历堆中的对象并清除其中的垃圾，整个过程需要标记对象的存活状态，用户程序在垃圾收集的过程中也不能执行，我们需要用到更复杂的机制来解决 STW 的问题。

我们对垃圾回收的印象，当垃圾内存到达某个阙值的时候，就会暂停整个程序，进行垃圾回收。Go 就是这么做的，为了提升性能，一直在优化，尽量缩短 STW 的时间。

#### 三色标记-清除

为了解决原始标记清楚算法带来的长时间 STW，多数垃圾收集器实现了三色标记算法的变种以缩短 STW 的时间。三色标记法算法将程序中的对象分成白色、黑色、灰色。

- 白色对象：潜在的垃圾，其内存可能会被垃圾收集器回收。
- 黑色对象：活跃的对象，根对象可达的对象以及不存在任何引用外部指针的对象。
- 灰色对象：活跃的对象，因为存在指向白色对象的外部指针，垃圾收集器会扫描这些对象的子对象。

在垃圾收集器开始工作时，程序中不存在任何的黑色对象，垃圾收集的根对象会被标记成灰色，垃圾收集器只从灰色对象集合中取出对象开始扫描，当灰色集合中不存在任何对象时，标记阶段就会结束。

当三色标记清除的标记阶段结束之后，应用程序的堆中就不存在任何的灰色对象，我们只能看到黑色存活的对象以及白色垃圾对象，垃圾收集器可以回收这些白色的垃圾。

因为用户程序可能在标记执行的过程中修改对象的指针，所以三色标记清除算法本身是不可以并发或者增量执行的，它仍然需要 STW。

#### 写屏障

想要在并发或者增量中保证标记的正确性，我们需要达成以下两种三色不变性中的一种。

- 强三色不变性：黑色对象不指向白色对象，只指向黑色或灰色。
- 弱三色不变性：黑色对象指向白色对象，该白色对象向上追溯，能够追溯到灰对象。

垃圾收集中的屏障技术更像一个钩子方法，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，根据操作类型的不同，我们可以将它们分成读屏障和写屏障。因为读屏障需要在读操作中加入代码片段，对用户程序的性能影响很大，所以编程语言往往采用写屏障保证三色不变性。

Go 语言中使用了两种写屏障：插入写屏障，删除写屏障。

插入写屏障，尝试修改指向指针的颜色，如果被指向的指针颜色是白色，则改成灰色，其他不变。满足强三色不变性。举个例子：

1. 垃圾收集器将根对象指向 A 对象标记成黑色，并将 A 指向的 B 标记成灰色；
2. 用户程序修改 A 对象的指针，将原本指向 B 的，改成 C，这时触发写屏障将 C 对象标出灰色；
3. 垃圾收集器依次遍历程序中其他的灰色对象，将它们标记为黑色。

删除写屏障，尝试修改被解除指向指针的颜色，如果被解除指针指针的颜色是白色，则改成灰色。满足弱不变性。举个例子：

1. 垃圾收集器根对象指向 A 对象标记成黑色，并将 A 指向的 B 标记成灰色；
2. 用户程序修改 A 对象指针，将原本指向 B 的，改成 C，触发删除写屏障，B 已被标记为灰色，则保持不变。
3. 用户程序修改 B 对象指针，将其指向 C，触发删除写屏障，C 为白色，则改成灰色。
4. 垃圾收集器依次遍历程序中其他的灰色对象，将他们标记为黑色。

#### 小结

v1.3 采用普通标记清除法，整体过程需要启动 STW，效率极低。

v1.5 采用三色标记法，堆空间启动写屏障，栈空间不启动，全部扫描之后，需要重新扫描一次栈（需要 STW），效率普通。

v1.8 采用三色标记法，混合写屏障机制，栈空间不启动，堆空间启动。整个过程几乎不需要 STW，效率较高。

#### 参考资料

https://zhuanlan.zhihu.com/p/334999060

## GMP 调度器原理

Go 语言的调度器用于线程上分配 Goroutine。

G 代表一个 Goroutine，存储了 goroutine 执行的栈信息，goroutine 的状态以及 goroutine 的任务函数等。

M 对应内核线程

P 相当于处理器拥有 G 队列，mcache 等供 goroutine 运行的资源。

除此之外还有一个全局 G 队列。

每个 G 对应一个系统线程 M 和其被分配的逻辑处理器核心 P。

**GPM 模式设计的策略**：

- 复用线程，避免频繁的创建，销毁线程，有两个机制：
  - work stealing 机制，当本地线程没有可运行的 G 时，从其他 P 本地 G 队列或全局 G 队列去偷。
  - hand off 机制，当本地线程阻塞后，会将 P 交给空闲线程或新建线程去处理。
- 利用并行，GOMAXPROCS 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。
- 抢占式调度，为了防止其他 goroutine 被饿死，一个 goroutinue 最多占用 CPU 10ms。
- 全局 G 队列，当执行 work stealing 的时候，其他 P 没有，则去全局队列获取。

**go func() 的执行过程**

1. 我们通过 go func() 来创建一个 goroutine；
2. 新创建的 goroutine 优先放到 P 的本地 G 队列里，如果满了，则放到全局队列里；
3. G 是由 M 运行的，一个 M 必须持有一个 P。M 会从 P 的本地 G 队列弹出一个可执行状态的 G 来执行，如果本地 G 队列为空，就会从其他的 MP 组合偷一个可执行的 G 执行；
4. 一个 M 调度 G 的过程是一个循环过程；
5. 当 M 执行某一个 G 的时候，如果发生阻塞，M 会阻塞，此时 P 如果还有可执行状态的 G 的话，P 会跟 M 断开，然后由去获取一个空线的 M 去服务 P ，如果没有空闲的 M，则由操作系统创建一个。
6. 当 M 系统调用结束后，G 会尝试获取一个空闲的 P，来执行。如果没有，则将 G 放到全局 G 队列，M 标记为空闲状态，加入空闲线程中。

## Channel 

Channels are inherently interesting

- Goroutine-safe
- Store and pass values between goroutines
- provide FIFO semantics
- can cause goroutines to block and unblock

#### making channels(the hchan struct)

```go
type hchan struct {
    buf    unsafe.Pointer
    sendx  uint
    recvx  uint
    sendq  waitq
    recvq  waitq
    lock   mutex
    ......
}

sendq -> sudog

type sudog struct {
    G	// waiting G
    elem // elem to send/recv
}
```

channel 在堆上分配的，ch := make(chan int, 3) 返回的是一个指针，所以 channel 在函数之间传递过程中不需要指针。

```
G1

ch <- tash4
gopart
calls into the scheduler
sets G1 to waiting
removes association between G1, M.

如何恢复 blocked goroutine?
the hchan struct stores waiting senders, receivers as well.
stores as a sudog, represents a G in a waiting list

create a sudog for itself, puts it in the sendq happens before calling into the scheduler
(receiver use it to resume G1)
```



```
G2

t := <- ch

goready(G1)
calls into scheduler
set G1 to runnable
puts it on runqueue
returns to G2


receive on an empty channel G2's execution is paused resumed after a send
set up state for resumption, and pause
set up state: put a sudog in the recvq
pause: gopark G2

```



sends and receives(goroutine scheduling)

stepping back(design considerations)



**goroutine-safe**

hchan struct

**store values, pass in FIFO**

copying into and out of hchan buffer

**can cause goroutines to pause and resume**

Hchan sudog queues

Calls into the scheduler(gopark, goready)

**unbuffer channel 没看懂**

unbuffer channels always work like the "direct send" case:

- Receiver first -> sender writes to receiver's stack
- Sender first -> receiver receives directly from the sudog.

**select(general-case)**

- all channels locked
- a sudog is put in the sendq/recvq queues of all channels
- Channels unlocked, and the select-ing G is paused
- CAS operation so there's one winning case
- resuming mirrors the pause sequeue 



接受的时候，数据是 copy 的什么意思？

```go
type sudog struct {
    G // Goroutine
    t // t：the memory location for the receive
}

G1 writes to t directly.
```

疑问：channel 在接受的时候都加锁释放锁。

#### 参考资料

https://www.youtube.com/watch?v=KBZlN0izeiY&t=13s

## Sync

#### 条件变量

条件变量并不是被用来保护临界区和共享资源的，它是用于协调想要访问共享资源的那些线程的。当共享资源的状态发生变化时，它可以用来通知被互斥锁阻塞的线程的。

```go
var lock = new(sync.Mutex)
var cond = sync.NewCond(locker)
```

Cond 有三个函数：

- Wait：等待通知
- Signal：通知阻塞的goroutine，可以执行了。
- Broadcast：广播，让所有阻塞的 goroutine 执行。

条件变量和锁，在并发时，如果逻辑不严谨，很容易造成死锁，故推荐 WaitGroup。

#### WaitGroup

```go
wg := sync.WaitGroup{}
for i := 0; i < 5; i++ {
  	wg.Add(1)
  	go func(i int, wg *sync.WaitGroup){
      	defer wg.Done()
      	fmt.Printf("%d starting\n", i)
      	time.Sleep(time.Second)
      	fmt.Printf("%d done\n", i)
    }(i, &wg)
}
wg.Wait()
```

一个可以帮我们实现一对多 goroutine 协作流程工具。

在使用 WaitGroup 的时候，我们最好先统一 Add，再并发 Done，最后 Wait。

#### Once

在众多 goroutine 里，只执行一次。

## Context

用处：控制 goroutine 的生命周期，比如：通过时间，信号等。

context.Context 有 Backgroup(), TODO(), WithVaule 方法，同包下还有 WitchCancel, WithDeadline, WithTimeout

context.Context 主要作用还是在多个 Goroutine 组成的树中同步信号以减少对资源的占用和消耗。虽然有传值的功能，但是很少用，大多用于分布式追踪的请求 ID。

## 程序性能分析

Go 语言为程序开发者提供了丰富的性能分析 API，这些 API 主要存在于：

- runtime/pprof
- net/http/pprof
- runtime/trace

在 Go 语言中，用于分析程序性能的概要文件有三种：CPU Profile，Mem Profile，Block Profile。
