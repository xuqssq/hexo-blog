---
title: Golang题库（六）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 40874
date: 2023-06-26 19:17:17
updated:
keywords:
description:
top_img:
comments:
cover:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---

# Golang题库（六）

## ✨讲一讲 GMP 模型

**三个字母的含义**

- `G（Goroutine）`：G 就是我们所说的 Go 语言中的协程 Goroutine 的缩写，相当于操作系统中的进程控制块。其中存着 goroutine 的运行时栈信息，CPU 的一些寄存器的值以及执行的函数指令等。
- `M（Machine）`：代表一个操作系统的主线程，对内核级线程的封装，数量对应真实的 CPU 数。一个 M 直接关联一个 os 内核线程，用于执行 G。M 会优先从关联的 P 的本地队列中直接获取待执行的 G。M 保存了 M 自身使用的栈信息、当前正在 M上执行的 G 信息、与之绑定的 P 信息。
- `P（Processor）`：Processor 代表了 M 所需的上下文环境，代表 M 运行 G 所需要的资源。是处理用户级代码逻辑的处理器，可以将其看作一个局部调度器使 go 代码在一个线程上跑。当 P 有任务时，就需要创建或者唤醒一个系统线程来执行它队列里的任务，所以 P 和 M 是相互绑定的。总的来说，P 可以根据实际情况开启协程去工作，它包含了运行 goroutine 的资源，如果线程想运行 goroutine，必须先获取 P，P 中还包含了可运行的 G 队列。

**源码**

1. **G**

```go
type g struct {
  stack       stack   		// 描述真实的栈内存，包括上下界

  m              *m     	// 当前的 m
  sched          gobuf   	// goroutine 切换时，用于保存 g 的上下文      
  param          unsafe.Pointer // 用于传递参数，睡眠时其他 goroutine 可以设置 param，唤醒时该goroutine可以获取
  atomicstatus   uint32
  stackLock      uint32 
  goid           int64  	// goroutine 的 ID
  waitsince      int64 		// g 被阻塞的大体时间
  lockedm        *m     	// G 被锁定只在这个 m 上运行
}
```

其中 sched 比较重要，该字段保存了 goroutine 的上下文。goroutine 切换的时候不同于线程有 OS 来负责这部分数据，而是由一个 gobuf 结构体来保存，gobuf 的结构如下：

```go
type gobuf struct {
    sp   uintptr
    pc   uintptr
    g    guintptr
    ctxt unsafe.Pointer
    ret  sys.Uintreg
    lr   uintptr
    bp   uintptr // for GOEXPERIMENT=framepointer
}
```

这里可以看出该结构体保存了当前的栈指针，计数器，还有 g 自身，这里记录自身 g 的指针的目的是为了**能快速的访问到 goroutine 中的信息**。

1. **M**

```go
type m struct {
    g0      *g     				// 带有调度栈的goroutine

    gsignal       *g         	// 处理信号的goroutine
    tls           [6]uintptr 	// thread-local storage
    mstartfn      func()
    curg          *g       		// 当前运行的goroutine
    caughtsig     guintptr 
    p             puintptr 		// 关联p和执行的go代码
    nextp         puintptr
    id            int32
    mallocing     int32 		// 状态

    spinning      bool 			// m是否out of work
    blocked       bool 			// m是否被阻塞
    inwb          bool 			// m是否在执行写屏蔽

    printlock     int8
    incgo         bool
    fastrand      uint32
    ncgocall      uint64      	// cgo调用的总数
    ncgo          int32       	// 当前cgo调用的数目
    park          note
    alllink       *m 			// 用于链接allm
    schedlink     muintptr
    mcache        *mcache 		// 当前m的内存缓存
    lockedg       *g 			// 锁定g在当前m上执行，而不会切换到其他m
    createstack   [32]uintptr 	// thread创建的栈
}
```

结构体 M 中，有两个重要的字段：

- curg：代表结构体M当前绑定的结构体 G 。
- g0 ：是带有调度栈的 goroutine，普通的 goroutine 的栈是在**堆上**分配的可增长的栈，但是 g0 的栈是 **M 对应的线程**的栈。与调度相关的代码，会先切换到该 goroutine 的栈中再执行。

1. **P**

```go
type p struct {
    lock mutex

    id          int32
    status      uint32 		// 状态，可以为pidle/prunning/...
    link        puintptr
    schedtick   uint32     // 每调度一次加1
    syscalltick uint32     // 每一次系统调用加1
    sysmontick  sysmontick 
    m           muintptr   // 回链到关联的m
    mcache      *mcache
    racectx     uintptr

    goidcache    uint64 	// goroutine的ID的缓存
    goidcacheend uint64

    // 可运行的goroutine的队列
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr

    runnext guintptr 		// 下一个运行的g

    sudogcache []*sudog
    sudogbuf   [128]*sudog

    palloc persistentAlloc // per-P to avoid mutex

    pad [sys.CacheLineSize]byte
}
```

- P 的个数就是 GOMAXPROCS（最大256），启动时固定的，一般不修改；GOMAXPOCS 默认值是当前电脑的核心数，单核CPU就只能设置为1，如果设置>1，在 GOMAXPOCS 函数中也会被修改为1。
- M 的个数和P 的个数不一定一样多（会有休眠的M或者不需要太多的 M）（M 最大10000）；
- 每一个 P 保存着本地 G 任务队列，也有一个全局 G 任务队列。

**模型介绍**

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/1648716696731-0d4e04b1-da32-4ff8-ac44-f2f74cfb7629-16488810664042.jpeg)
本地队列：存放等待运行的 G，一个本地队列存放的G数量一般不超过 256 个，优先将新创建的 G 放在 P 的本地队列中，如果满了会放在全局队列中。
全局队列：存放等待运行的 G，读写要**加锁**，所以拿取效率在多线程竞争的情况下相比于本地队列来说要低。

**面试回答模板**

首先呢，GMP 这三个字母的含义分别是 Goroutine，Machine，Processor。这个Goroutine，相当于操作系统中的进程控制块。其中存着 goroutine 的运行时栈信息，CPU 的一些寄存器的值以及执行的函数指令等。Machine就是代表了一个操作系统的主线。M 结构体中，保存了 M 自身使用的栈信息、当前正在 M上执行的 G 信息、与之绑定的 P 信息。M 直接关联一个 os 内核线程，用于执行 G。（这里思考一个这个模型的图片回答），这个 M 做的事情就是从关联的 P 的本地队列中直接获取待执行的 G。剩下的 Processor 是代表了 M 所需的上下文环境，代表 M 运行 G 所需要的资源。当 P 有任务时，就需要创建或者唤醒一个系统线程来执行它队列里的任务。在GMP调度模型中，P 的个数就是 GOMAXPROCS，是可以手动设置的，但一般不修改，GOMAXPOCS 默认值是当前电脑的核心数，单核CPU就只能设置为1，如果设置>1，在 GOMAXPOCS 函数中也会被修改为1。总的来说，这个 P 结构体的主要的任务就是可以根据实际情况开启协程去工作。

## 🌟了解的GC算法有哪些？

常见的垃圾回收算法有以下几种：

### 引用计数

**引用计数：**对每个对象维护一个引用计数，当引用该对象的对象被销毁时，引用计数减1，当引用计数器为0时回收该对象。
优点：对象可以很快的被回收，不会出现内存耗尽或达到某个阀值时才回收。
缺点：不能很好的处理循环引用，而且实时维护引用计数，有也一定的代价。
`代表语言：Python、PHP`

### 标记-清除

**标记-清除：**从根变量开始遍历所有引用的对象，引用的对象标记为"被引用"，没有被标记的进行回收。
优点：解决了引用计数的缺点。
缺点：需要STW，即要暂时停掉程序运行。
`代表语言：Golang(其采用三色标记法)`

### 分代收集

**分代收集：**按照对象生命周期长短划分不同的代空间，生命周期长的放入老年代，而短的放入新生代，不同代有不能的回收算法和回收频率。
优点：回收性能好
缺点：算法复杂
`代表语言： JAVA`

### **三色标记法**

1. 初始状态下所有对象都是白色的。

2. 从根节点开始遍历所有对象，把遍历到的对象变成灰色对象
3. 遍历灰色对象，将灰色对象引用的对象也变成灰色，然后将遍历过的灰色对象变成黑色对象。
4. 循环步骤3，直到灰色对象全部变黑色。
5. 回收所有白色对象（垃圾）。

## go垃圾回收，什么时候触发

### 主动触发

主动触发(手动触发)，通过调用 runtime.GC 来触发GC，此调用阻塞式地等待当前GC运行完毕。

### 被动触发

被动触发，分为两种方式：

1. 使用**步调（Pacing）算法**，其核心思想是控制内存增长的比例,每次内存分配时检查当前内存分配量是否已达到**阈值（环境变量GOGC）：默认100%**，即**当内存扩大一倍时启用GC**。
2. 使用系统监控，当**超过两分钟**没有产生任何GC时，强制触发 GC。

## 深拷贝和浅拷贝

### 深拷贝

**拷贝的是数据本身**，创造一个新对象，新创建的对象与原对象不共享内存，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值。

### 浅拷贝

**拷贝的是数据地址**，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化。
实现浅拷贝的方式：引用类型的变量,默认赋值操作就是浅拷贝

## 为什么不要大量使用goroutine

**合理复用协程**

使用goroutine可以帮助提高程序的并发性和性能，但是过度使用goroutine会带来一些问题，例如：

- 内存占用增加，因为每个goroutine都需要占用一定的内存
- 过多的goroutine会导致CPU上下文切换频繁，影响程序性能
- 如果goroutine没有正确的管理，可能会导致资源泄漏或死锁
  为了优化这些问题，可以考虑以下方法：
- 确定适当的goroutine数量，避免过度使用goroutine。
- 使用有限的goroutine池，以限制goroutine的总数，并避免内存占用过多。
- 优化goroutine的调度，以减少CPU上下文切换的次数。
- 使用通道和其他同步原语来避免竞争条件和死锁。

## channel有缓冲和无缓冲在使用上有什么区别？

无缓冲：发送和接收需要同步。
有缓冲：不要求发送和接收同步，缓冲满时发送阻塞。
因此 channel 无缓冲时，发送阻塞直到数据被接收，接收阻塞直到读到数据；channel有缓冲时，当缓冲满时发送阻塞，当缓冲空时接收阻塞。

## go 的优势

1. 与其他作为学术实验开始的语言不同，Go 代码的设计是务实的。每个功能和语法决策都旨在让程序员的生活更轻松。

2. Golang针对并发进行了优化，并且在规模上运行良好。
3. 由于单一的标准代码格式，Golang 通常被认为比其他语言更具可读性。
4. 自动垃圾收集明显比 Java 或 Python 更有效，因为它与程序同时执行。
5. Go在语言层面支持高并发
6. Go属于开发效率和运行效率折中的一门语言

## 如何判断channel是否关闭？

- 读channel的时候判断其是否已经关闭
  `_,ok := <- jobs`
  此时如果 channel 关闭，ok 值为 false
  
- 写入channel的时候判断其是否已经关闭

  1. `_,ok := <- jobs`

     此时如果 channel 关闭，ok 值为 false，如果 channel 没有关闭，则会漏掉一个 jobs中的一个数据

  2. 使用 select 方式

     再创建一个 channel，叫做 timeout，如果超时往这个 channel 发送 true，在生产者发送数据给 jobs 的 channel，用 select 监听 timeout，如果超时则关闭 jobs 的 channel。

## make 与 new 的区别

**引用类型与值类型**

`引用类型` 变量存储的是一个地址，这个地址存储最终的值。内存通常在堆上分配。通过 GC 回收。包括 指针、slice 切片、管道 channel、接口 interface、map、函数等。

`值类型`是 基本数据类型，int,float,bool,string, 以及数组和 struct 特点：变量直接存储值，内存通常在栈中分配，栈在函数调用后会被释放

对于`引用类型`的变量，我们不光要声明它，还要为它分配内容空间

对于`值类型`的则不需要显示分配内存空间，是因为go会默认帮我们分配好

**new()**

```
func new(Type) *Type
```

new()对类型进行内存分配,入参为类型,返回为类型的指针，指向分配类型的内存地址

**make()**

```
func make(t Type, size ...IntegerType) Type
```

make()也是用于内存分配的，但是和new不同，它只用于channel、map以及切片的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

注意，因为这三种类型是引用类型，所以必须得初始化，但是不是置为零值，这个和new是不一样的。

简而言之make()用于初始化slice, map, channel等内置数据结构
