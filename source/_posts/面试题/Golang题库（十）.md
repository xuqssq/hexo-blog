---
title: Golang题库（十）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 52804
date: 2023-07-05 22:11:49
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

# Golang题库（十）

## 空结构体占不占内存空间？ 为什么使用空结构体？

### **空结构体是没有内存大小的结构体。**

通过 `unsafe.Sizeof()` 可以查看空结构体的宽度，代码如下：

```go
var s struct{}
fmt.Println(unsafe.Sizeof(s)) // prints 0
```

准确的来说，空结构体有一个特殊起点： `zerobase` 变量。`zerobase`是一个占用 8 个字节的`uintptr`全局变量。每次定义 `struct {}` 类型的变量，编译器只是把`zerobase`变量的地址给出去。也就是说空结构体的变量的内存地址都是一样的。

### 空结构体的使用场景主要有三种：

- 实现方法接收者：在业务场景下，我们需要将方法组合起来，代表其是一个 ”分组“ 的，便于后续拓展和维护。
- 实现集合类型：在 Go 语言的标准库中并没有提供集合（Set）的相关实现，因此一般在代码中我们图方便，会直接用 map 来替代：`type Set map[string]struct{}`。
- 实现空通道：在 Go channel 的使用场景中，常常会遇到通知型 channel，其不需要发送任何数据，只是用于协调 Goroutine 的运行，用于流转各类状态或是控制并发情况。

## Kratos 框架的特性

Kratos 是一套轻量级的微服务框架，包含了大量微服务相关框架以及工具，它就像一个工具箱，目前已加入 CNCF 协会进行孵化。Kratos 框架最重要的特性就是可插拔，它并没有像字节的 go 微服务框架一样打造一套属于自己的生态，而是选择依赖开源社区的明星项目，将它们灵活的集成到 Kratos 中。

## defer 是怎么用的

### 从 defer 关键字的常见使用场景和使用时需要注意什么来回答这个问题（不深入到实现原理）

**defer 最常见的使用场景就是在函数调用结束后，完成一些收尾工作**，例如在 defer 中回滚数据库的事务。在 go 语言中使用 defer 常会遇到的两个问题，首先是 defer 关键字的调用时机， defer 被多次调用时的执行顺序，其次是 defer 使用传值的方式传递参数时会进行预计算，会导致结果不符合预期。调用时机与作用域有关，预计算参数与预期不符与 defer 关键字的复制操作有关。

## Context 包的作用

Context 就像糖葫芦中的竹签子
它的作用是在上下文中传递除了业务参数之外的额外信息，这个额外信息是为了全局而考虑使用的，例如在微服务业务中，我们需要整个业务链条整体的超时时间信息。不过 go 标准库中的 Context 还提供了超时 Timeout 和 Cancel 机制。总的来说，在下面这些场景中，可以考虑使用 Context：

- 上下文信息传递
- 控制子 goroutine 的运行
- 超时控制的方法调用
- 可以取消的方法调用

## golang并发模型

[参考文章](https://studygolang.com/articles/10631?fr=sidebar)

golang控制并发有三种经典的方式,一种是通过**channel**通知实现并发控制 一种是**WaitGroup**,另外一种就是**Context**。

1. 使用最基本通过channel通知实现并发控制
   无缓冲通道:
   无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送 goroutine 和接收 goroutine 同时准备好，才可以完成发送和接收操作。
   从上面无缓冲的通道定义来看，发送 goroutine 和接收 gouroutine 必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相对应的操作准备好为止。这种无缓冲的通道我们也称之为同步通道。
   例子:

   ```go
    func main() {
        ch := make(chan struct{})
        go func() {
            ch <- struct{}{}
        }()
        fmt.Println(<-ch)
    }
   ```

   **解释**
   当主 goroutine 运行到 <-ch 接受 channel 的值的时候，如果该 channel 中没有数据，就会一直阻塞等待，直到有值。 这样就可以简单实现并发控制。

2. 通过sync包中的WaitGroup实现并发控制
   在 sync 包中，提供了 WaitGroup ，它会等待它收集的所有 goroutine 任务全部完成，在主 goroutine 中 Add(delta int) 索要等待goroutine 的数量。
   在每一个 goroutine 完成后 Done() 表示这一个goroutine 已经完成，当所有的 goroutine 都完成后，在主 goroutine 中 WaitGroup 返回返回。

   例子：

   ```go
   fun main(){
       var wg sync.WaitGroup
       var urls = []string{
           "http://www.golang.org/",
           "http://www.sensetime.com/",
           "http://www.baidu.com/",
       }
       for _, url := range urls {
           wg.Add(1)
           go func(url string) {
               defer wg.Done()
               http.Get(url)
           }(url)
       }
       wg.Wait()
   }
   ```

3. 在Go 1.7 以后引进的强大的Context上下文，实现并发控制
   在一些简单场景下使用 channel 和 WaitGroup 已经足够了，但是当面临一些复杂多变的网络并发场景下 channel 和 WaitGroup 显得有些力不从心了。
   比如一个网络请求 Request，每个 Request 都需要开启一个 goroutine 做一些事情，这些 goroutine 又可能会开启其他的 goroutine，比如数据库和RPC服务。
   所以我们需要一种可以跟踪 goroutine 的方案，才可以达到控制他们的目的，这就是Go语言为我们提供的 Context，称之为上下文非常贴切，它就是goroutine 的上下文。
   它是包括一个程序的运行环境、现场和快照等。每个程序要运行时，都需要知道当前程序的运行状态，通常Go 将这些封装在一个 Context 里，再将它传给要执行的 goroutine 。
   context 包主要是用来处理多个 goroutine 之间共享数据，及多个 goroutine 的管理。
   context包方法:
   Done() 返回一个只能接受数据的channel类型，当该context关闭或者超时时间到了的时候，该channel就会有一个取消信号
   Err() 在Done() 之后，返回context 取消的原因。
   Deadline() 设置该context cancel的时间点
   Value() 方法允许 Context 对象携带request作用域的数据，该数据必须是线程安全的。
   例子:

   ```go
   func childFunc(cont context.Context, num *int) {
       ctx, _ := context.WithCancel(cont)
       for {
           select {
           case <-ctx.Done():
               fmt.Println("child Done : ", ctx.Err())
               return
           }
       }
   }
   
   func main() {
       gen := func(ctx context.Context) <-chan int {
           dst := make(chan int)
           n := 1
           go func() {
               for {
                   select {
                   case <-ctx.Done():
                       fmt.Println("parent Done : ", ctx.Err())
                       return // returning not to leak the goroutine
                   case dst <- n:
                       n++
                       go childFunc(ctx, &n)
                   }
               }
           }()
           return dst
       }
   
       ctx, cancel := context.WithCancel(context.Background())
       for n := range gen(ctx) {
           fmt.Println(n)
           if n >= 5 {
               break
           }
       }
       cancel()
       time.Sleep(5 * time.Second)
   }
   ```

   在上面的例子中，主要描述的是通过一个channel实现一个为循环次数为5的循环，在每一个循环中产生一个goroutine，每一个goroutine中都传入context，在每个goroutine中通过传入ctx创建一个子Context,并且通过select一直监控该Context的运行情况，当在父Context退出的时候，代码中并没有明显调用子Context的Cancel函数，但是分析结果，子Context还是被正确合理的关闭了，这是因为，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅的解决了goroutine启动后不可控的问题。

## ⭐golang gmp模型，全局队列中的G会不会饥饿,为什么？P的数量是多少？能修改吗？M的数量是多少？

全局队列中的G不会饥饿，P中每执行61次调度，就需要优先从全局队列中获取一个G到当前P中，并执行下一个要执行的G。

调度协程的优先级与顺序：
![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20220403162913.png)

P，可以通过 `runtime.GOMAXPROCS()` 设置数量，默认为当前CP

**M数量问题**
Go语⾔本身是限定M的**最⼤量是10000**。
`runtime/debug`包中的SetMaxThreads函数来设置。
有⼀个M阻塞，会创建⼀个新的M。
如果有M空闲，那么就会回收或者睡眠。

## go 语言的 panic 如何恢复

recover()
recover和panic必须在同一个goroutine中
recover必须放到延迟执行函数defer中

## defer 的执行顺序

在同一个函数中，defer 函数调用的执行顺序与它们分别所属的 defer 语句的出现顺序完全相反。当一个函数即将结束执行时，写在最下面的 defer 函数调用会最先执行，其次是写在他上边，与它的距离最近的那个 defer 函数调用，以此类推，最上面的 defer 函数调用会最后一个执行。
需要注意一下 for 循环中的 defer 执行顺序。如果函数中有一条 for 循环语句，并且这个 for 循环语句中包含了一条 defer 语句，那么 defer 语句的执行是怎样的？弄清楚这个问题需要弄明白 defer 语句执行时发生的事情。在 defer 语句每次执行的时候，go 语言会把它携带的 defer 函数及其参数值存储到一个链表中，这个链表叫做 goroutine_defer。这个链表与 defer 语句所属的函数是对应的，它是先进先出的，相当于一个栈。在执行某个函数中的 defer 函数调用的时候，go 语言会先拿到对应的链表，然后从链表中一个一个取出 defer 函数及其参数值，逐个调用，这也就是为什么说 “defer 函数调用的执行顺序与它们分别所属的 defer 语句的出现顺序完全相反”。

## 服务器能开多少个M由什么决定

- 由于M必须持有一个P才可以运行Go代码，所以同时运行的M个数，也即线程数一般等同于CPU的个数，以达到尽可能的使用CPU而又不至于产生过多的线程切换开销。
- P的个数默认等于CPU核数，每个M必须持有一个P才可以执行G，一般情况下M的个数会略大于P的个数，这多出来的M将会在G产生系统调用时发挥作用。
- Go语⾔本身是限定M的最⼤量是10000，可以在runtime/debug包中的SetMaxThreads函数来修改设置

## 服务器能开多少个P由什么决定

- P的个数在程序启动时决定，默认情况下等同于CPU的核数
- 程序中可以使用 runtime.GOMAXPROCS() 设置P的个数，在某些IO密集型的场景下可以在一定程度上提高性能。
- 一般来讲，程序运行时就将GOMAXPROCS大小设置为CPU核数，可让Go程序充分利用CPU。在某些IO密集型的应用里，这个值可能并不意味着性能最好。理论上当某个Goroutine进入系统调用时，会有一个新的M被启用或创建，继续占满CPU。但由于Go调度器检测到M被阻塞是有一定延迟的，也即旧的M被阻塞和新的M得到运行之间是有一定间隔的，所以在IO密集型应用中不妨把GOMAXPROCS设置的大一些，或许会有好的效果。

## M和P是怎么样的关系

M必须拥有P才可以执行G中的代码，理想情况下一个M对应一个P，P含有包含多个G的队列，P会周期性地将G调度到M种执行。

## 同时启动了一万个G，如何调度？

首先一万个G会按照P的设定个数，尽量平均地分配到每个P的本地队列中。如果所有本地队列都满了，那么剩余的G则会分配到GMP的全局队列上。接下来便开始执行GMP模型的调度策略：

- **本地队列轮转**：每个P维护着一个包含G的队列，不考虑G进入系统调用或IO操作的情况下，P周期性的将G调度到M中执行，执行一小段时间，将上下文保存下来，然后将G放到队列尾部，然后从队首中重新取出一个G进行调度。
- **系统调用**：上面说到P的个数默认等于CPU核数，每个M必须持有一个P才可以执行G，一般情况下M的个数会略大于P的个数，这多出来的M将会在G产生系统调用时发挥作用。当该G即将进入系统调用时，对应的M由于陷入系统调用而进被阻塞，将释放P，进而某个空闲的M1获取P，继续执行P队列中剩下的G。
- **工作量窃取**：多个P中维护的G队列有可能是不均衡的，当某个P已经将G全部执行完，然后去查询全局队列，全局队列中也没有新的G，而另一个M中队列中还有3很多G待运行。此时，空闲的P会将其他P中的G偷取一部分过来，一般每次偷取一半。

## go的init函数是什么时候执行的？

- init函数的主要作用：
  1）初始化不能采用初始化表达式初始化的变量。
  2）程序运行前的注册。
  3）实现sync.Once功能。
  4）其他
- init函数的主要特点：
  1）init函数先于main函数自动执行，不能被其他函数调用；
  2）init函数没有输入参数、返回值；
  3）每个包可以有多个init函数；
  4）包的每个源文件也可以有多个init函数，这点比较特殊；
  5）同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。
  6）不同包的init函数按照包导入的依赖关系决定执行顺序。
- golang程序初始化
  golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：
  1）初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见golang变量的初始化）；
  2）初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化，参见golang变量的初始化）；
  3）执行包的init函数；

**故，最终初始化顺序:变量初始化 -> init() -> main()**
