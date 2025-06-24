---
title: Golang题库（二）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 30645
date: 2023-06-20 18:44:21
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

# Golang题库（二）

## 数组怎么转集合?

无法直接转换，需要通过遍历数组，构造一个map。例如：

```go
func main() {
	arr := [5]int{1, 2, 3, 4, 5}
	m := make(map[int]int, 5)
	for i, v := range arr {
		m[i] = v
	}
	fmt.Println(m)
}
```

## ⭐Go的GMP模型?

G是`Goroutine`的缩写，相当于操作系统的进程控制块(process control block)。它包含：函数执行的指令和参数，任务对象，线程上下文切换，字段保护，和字段的寄存器。

M是一个线程，每个M都有一个线程的栈。如果没有给线程的栈分配内存，操作系统会给线程的栈分配默认的内存。当线程的栈制定，M.stack->G.stack, M的PC寄存器会执行G提供的函数。

P(处理器，Processor)是一个抽象的概念，不是物理上的CPU。当一个P有任务，需要创建或者唤醒一个系统线程去处理它队列中的任务。P决定同时执行的任务的数量，`GOMAXPROCS`限制系统线程执行用户层面的任务的数量。

GO调度器的调度过程，首先创建一个G对象，然后G被保存在P的本地队列或者全局队列（global queue）。这时P会唤醒一个M。P按照它的执行顺序继续执行任务。M寻找一个空闲的P，如果找得到，将G与自己绑定。然后M执行一个调度循环：调用G对象->执行->清理线程->继续寻找Goroutine。

在M的执行过程中，上下文切换随时发生。当切换发生，任务的执行现场需要被保护，这样在下一次调度执行可以进行现场恢复。M的栈保存在G对象中，只有现场恢复需要的寄存器(SP,PC等)，需要被保存到G对象。

如果G对象还没有被执行，M可以将G重新放到P的调度队列，等待下一次的调度执行。当调度执行时，M可以通过G的vdsoSP, vdsoPC 寄存器进行现场恢复。

P队列 P有2种类型的队列：

1. 本地队列：本地的队列是无锁的，没有数据竞争问题，处理速度比较高。
2. 全局队列：是用来平衡不同的P的任务数量，所有的M共享P的全局队列。

线程清理 G的调度是为了实现P/M的绑定，所以线程清理就是释放P上的G，让其他的G能够被调度。

1. 主动释放(active release)：典型的例子是，执行G任务时，发生了系统调用(system call)，这时M会处于阻塞（Block）状态。调度器会设置一个超时时间，来释放P。
2. 被动释放(passive release)：如果系统调用发生，监控程序需要扫描处于阻塞状态的P/M。 这时，超时之后，P资源会回收，程序被安排给队列中的其他G任务。

## Go和java比有什么不同?

Go也称为Golang，是一种开源编程语言，Go可以轻松构建可靠，简单和高效的软件。Go是键入的静态编译语言。Go语言提供垃圾收机制，CSP风格的并发性，内存安全性和结构类型。

Java是一种用于一般用途的计算机编程语言，它是基于类的，并发的和面向对象的。Java专门设计为包含很少的实现依赖项。Java应用程序在JVM（Java虚拟机）上运行。它是当今最著名的编程语言之一。Java是一种用于为多个平台开发软件的编程语言。Java应用程序上的编译代码或字节码可以在大多数操作系统上运行，包括Linux，Mac操作系统和Linux。Java的大部分语法都源自C ++和C语言。

go语言和java之间的区别

- 函数重载

  Go上不允许函数重载，必须具有方法和函数的唯一名称；

  java允许函数重载。

- 速度

  go的速度比java快

- 多态

  Java默认允许多态。而Go没有。

- 路由配置

  Go语言使用HTTP协议进行路由配置；

  java使用Akka.routing.ConsistentHashingRouter和Akka.routing.ScatterGatherFirstCompletedRouter进行路由配置。

- 可扩展性

  Go代码可以自动扩展到多个核心；而Java并不总是具有足够的可扩展性。

- 继承

  Go语言的继承通过匿名组合完成：基类以Struct的方式定义，子类只需要把基类作为成员放在子类的定义中，支持多继承；

  Java的继承通过extends关键字完成，不支持多继承。

## 介绍一下channel

`Channel`是`Go`里面的一种并发原语，每一个管道对象都有一种具体的类型，例如`chan int`一种传输`int`类型的管道。

`chan`是一种在函数传输中是一种引用类型，同其他引用类型一样，零值为`nil`。

通道有两个主要操作：发送(send)和接收(receive)，两者统称为通信。send语句从一个goroutine传输一个值到另一个在执行接收表达式的goroutine。两个操作都使用`<-`操作符书写。发送语句中，通道和值分别在`<-`的左右两边。在接收表达式中，`<-`放在通道操作数前面，在接收表达式中，其结果未被使用也是合法的。

```go
ch <- x		//发送语句
x = <-ch	//接收语句
<-ch		//接收语句，丢弃结果
```

通道支持第三个操作：关闭 (close)，它设置一个标志位来指示值当前已经发送完毕，这个通道后面没有值了；关闭后的发送操作将导致宕机。在一个已经关闭的通道上进行接收操作，将获取所有已经发送的值，直到通道为空；这时任何接收操作会立即完成，同时获取到一个通道元素对应的零值。通过调用内置的`close`函数来关闭通道：

```go
close(ch)
```

根据通道的容量，可以将通道分为无缓冲通道和缓冲通道

- 无缓冲通道

  ```go
  ch = make(chan int)
  ch = make(chan int, 0)
  ```

- 有缓冲通道

  ```go
  ch = make(chan int, 3)
  ```

根据通道传输方向，还可以通道分为双向通道，只读通道和只写通道

- 只读通道

  只能发送的通道，允许发送但不允许接收

  ```go
  chan<- int
  ```

- 只写通道

  只能接收的通道，允许接收但不允许发送

  ```go
  <-chan int
  ```

## channel实现方式/原理/概念/底层实现

**背景：**

- Go语言提供了一种不同的并发模型–通信顺序进程(communicating sequential processes,CSP)。
- 设计模式：通过通信的方式共享内存
- channel收发操作遵循先进先出(FIFO)的设计

**底层结构:**

```go
type hchan struct {
    qcount   uint           // channel中的元素个数
    dataqsiz uint           // channel中循环队列的长度
    buf      unsafe.Pointer // channel缓冲区数据指针
    elemsize uint16            // buffer中每个元素的大小
    closed   uint32            // channel是否已经关闭，0未关闭
    elemtype *_type // channel中的元素的类型
    sendx    uint   // channel发送操作处理到的位置
    recvx    uint   // channel接收操作处理到的位置
    recvq    waitq  // 等待接收的sudog（sudog为封装了goroutine和数据的结构）队列由于缓冲区空间不足而阻塞的Goroutine列表
    sendq    waitq  // 等待发送的sudogo队列，由于缓冲区空间不足而阻塞的Goroutine列表

    lock mutex   // 一个轻量级锁
}
```

**channel创建:**

```go
ch := make(chan int, 3)
```

- 创建channel实际上就是在内存中实例化了一个***hchan***结构体，并返回一个chan指针
- channle在函数间传递都是使用的这个指针，这就是为什么函数传递中无需使用channel的指针，而是直接用channel就行了，因为channel本身就是一个指针

**channel发送数据：**

```go
ch <- 1
ch <- 2
```

- 检查 recvq 是否为空，如果不为空，则从 recvq 头部取一个 goroutine，将数据发送过去，并唤醒对应的 goroutine 即可。
- 如果 recvq 为空，则将数据放入到 buffer 中。
- 如果 buffer 已满，则将要发送的数据和当前 goroutine 打包成 sudog 对象放入到 sendq中。并将当前 goroutine 置为 waiting 状态。

**channel接收数据：**

```go
<-ch
<-ch
```

- 检查sendq是否为空，如果不为空，且没有缓冲区，则从sendq头部取一个goroutine，将数据读取出来，并唤醒对应的goroutine，结束读取过程。
- 如果sendq不为空，且有缓冲区，则说明缓冲区已满，则从缓冲区中首部读出数据，把sendq头部的goroutine数据写入缓冲区尾部，并将goroutine唤醒，结束读取过程。
- 如果sendq为空，缓冲区有数据，则直接从缓冲区读取数据，结束读取过程。
- 如果sendq为空，且缓冲区没数据，则只能将当前的goroutine加入到recvq,并进入waiting状态，等待被写goroutine唤醒。

**channel规则：**

| 操作        | 空channel | 已关闭channel | 活跃中的channel |
| ----------- | --------- | ------------- | --------------- |
| close(ch)   | panic     | panic         | 成功关闭        |
| ch<- v      | 永远阻塞  | panic         | 成功发送或阻塞  |
| v,ok = <-ch | 永远阻塞  | 不阻塞        | 成功接收或阻塞  |
