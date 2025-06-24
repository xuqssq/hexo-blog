---
title: Golang题库（八）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 6059
date: 2023-07-03 21:27:34
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

# Golang题库（八）

## go语言的引用类型有什么？

1. 切片(slice)类型
2. map类型 
3. 管道(channel)类型
4. 接口(interface)类型

## map的key可以是哪些类型？可以嵌套map吗？

1. key的类型
   - bool, 
   - int，
   - string,
   - 指针
   - channel 
   - interface 
   - structs
   - arrays 
2. map是可以进行嵌套的。

## 协程goroutine

协程是一种用户态的轻量级线程，协程的调度完全由用户控制（进程和线程都是由cpu 内核进行调度）。

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

对于进程、线程，都是有内核进行调度，有 CPU 时间片的概念，进行抢占式调度（有多种调度算法）。而对于协程(用户级线程)，这是对内核透明的，也就是系统并不知道有协程的存在，是完全由用户自己的程序进行调度的，因为是由用户程序自己控制，那么就很难像抢占式调度那样做到强制的 CPU 控制权切换到其他进程/线程，通常只能进行协作式调度，需要协程自己主动把控制权转让出去之后，其他协程才能被执行到。

goroutine和协程区别:
本质上，goroutine 就是协程。
不同的是，Golang 在 runtime、系统调用等多方面对 goroutine 调度进行了封装和处理，当遇到长时间执行或者进行系统调用时，会主动把当前 goroutine 的CPU § 转让出去，让其他 goroutine 能被调度并执行，也就是 Golang 从语言层面支持了协程。
Golang 的一大特色就是从语言层面原生支持协程，在函数或者方法前面加 go关键字就可创建一个协程。

## 讲一下set的原理，Java 的HashMap和 go 的map底层原理

**1. Set原理:**
Set特性: 1. 不包含重复key. 2.无序.
如何去重:
通过查看源码add(E e)方法，底层实现有一个map，map是HashMap,Hash类型是散列，所以是无序的.
如果key值相同，将会覆盖，这就是set为什么能去重的原因(key相同会覆盖).
**注意:**
如果new出两个对象add到set中,因为两个对象的地址不相同,所以map在计算key的hash值时，将它当成了两个不同的元素。这时要重写equals和hashcode两个方法。
这样才能保证set集合的元素不重复.

**2. Java HashMap:**

线程不安全 安全的map(CurrentHashMap)
HashMap由数组+链表组成,数组是HashMap的主体,
链表则是为了解决哈希冲突而存在的,如果定位到的数组位置不含链表（当前entry的next指向null）,那么查找，添加等操作很快，仅需一次寻址即可；
如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；
对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。
所以，性能考虑，HashMap中的链表出现越少，性能才会越好。
假如一个数组槽位上链上数据过多（即链表过长的情况）导致性能下降该怎么办？
JDK1.8在JDK1.7的基础上针对增加了红黑树来进行优化。
即当链表超过8时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。

**3. go map:**

线程不安全 安全的map(sync.map)
特性: 1. 无序. 2. 长度不固定. 3. 引用类型.
底层实现:
1.hmap 2.bmap(bucket)
hmap中含有n个bmap，是一个数组.
每个bucket又以链表的形式向下连接新的bucket.
bucket关注三个字段: 1. 高位哈希值 2. 存储key和value的数组 3. 指向扩容bucket的指针
高位哈希值: 用于寻找bucket中的哪个key.
低位哈希值: 用于寻找当前key属于hmap中的哪个bucket.
map的扩容:
当map中的元素增长的时候，Go语言会将bucket数组的数量扩充一倍，产生一个新的bucket数组，并将旧数组的数据迁移至新数组。
加载因子
判断扩充的条件，就是哈希表中的加载因子(即loadFactor)。
加载因子是一个阈值，一般表示为：散列包含的元素数 除以 位置总数。是一种“产生冲突机会”和“空间使用”的平衡与折中：加载因子越小，说明空间空置率高，空间使用率小，但是加载因子越大，说明空间利用率上去了，但是“产生冲突机会”高了。
每种哈希表的都会有一个加载因子，数值超过加载因子就会为哈希表扩容。
Golang的map的加载因子的公式是：map长度 / 2^B(这是代表bmap数组的长度，B是取的低位的位数)阈值是6.5。其中B可以理解为已扩容的次数。
当Go的map长度增长到大于加载因子所需的map长度时，Go语言就会将产生一个新的bucket数组，然后把旧的bucket数组移到一个属性字段oldbucket中。注意：并不是立刻把旧的数组中的元素转义到新的bucket当中，而是，只有当访问到具体的某个bucket的时候，会把bucket中的数据转移到新的bucket中。
map删除:
并不会直接删除旧的bucket，而是把原来的引用去掉，利用GC清除内存。

## ⭐go的GC（标记清理 -> 三色标记发 -> 混合写屏障）

1. **标记清除:**
   此算法主要有两个主要的步骤：

   标记(Mark phase)

   清除(Sweep phase)

   第一步，找出不可达的对象，然后做上标记。
   第二步，回收标记好的对象。

   操作非常简单，但是有一点需要额外注意：mark and sweep算法在执行的时候，需要程序暂停！即 stop the world。
   也就是说，这段时间程序会卡在哪儿。故中文翻译成 卡顿.

   标记-清扫(Mark And Sweep)算法存在什么问题？
   标记-清扫(Mark And Sweep)算法这种算法虽然非常的简单，但是还存在一些问题：

   STW，stop the world；让程序暂停，程序出现卡顿。

   标记需要扫描整个heap

   清除数据会产生heap碎片
   这里面最重要的问题就是：mark-and-sweep 算法会暂停整个程序。

2. **三色并发标记法:**
   首先：程序创建的对象都标记为白色。
   gc开始：扫描所有可到达的对象，标记为灰色
   从灰色对象中找到其引用对象标记为灰色，把灰色对象本身标记为黑色
   监视对象中的内存修改，并持续上一步的操作，直到灰色标记的对象不存在
   此时，gc回收白色对象
   最后，将所有黑色对象变为白色，并重复以上所有过程。

3. **混合写屏障:**
   注意：
   当gc进行中时，新创建一个对象. 按照三色标记法的步骤,对象会被标记为白色,这样新生成的对象最后会被清除掉，这样会影响程序逻辑.
   golang引入写屏障机制.可以监控对象的内存修改，并对对象进行重新标记.
   gc一旦开始，无论是创建对象还是对象的引用改变，都会先变为灰色。

## go 中用 for 遍历多次执行 goroutine会存在什么问题

`goroutine`并非按照`for`循环的顺序执行协程函数的，因为初始化协程也是消耗资源的，所以`go`的协程是并发执行的，且`for`循环的索引，是`goroutine`准备好再取读取对应的值，所以**可能出现多个`goroutine`获取到的`入参`都是同一个值。**

## ⭐gmp当一个g堵塞时，g、m、p会发生什么

当g阻塞时，p会和m解绑，去寻找下一个可用的m。
g&m在阻塞结束之后会优先寻找之前的p，如果此时p已绑定其他m，当前m会进入休眠，g以可运行的状态进入全局队列。

## ⭐（🐮啊，胖🐯）Golang 逃逸分析

面试官：“写过C/C++的同学都知道，调用著名的malloc和new函数可以在堆上分配一块内存，这块内存的使用和销毁的责任都在程序员。一不小心，就会发生内存泄露。那你说下Golang 是怎么处理这个问题的”

胖虎：“Golang 通过逃逸分析，对内存管理进行的优化和简化，它可以决定一个变量是分配到堆还栈上。”

**什么是golang的逃逸分析**

面试官：“那你说下什么是逃逸分析吧”

胖虎想：“这道题我会啊，准备好了吗，我要开始装X了。”

Golang 的逃逸分析，是指编译器根据代码的特征和生命周期，自动的把变量分配到堆或者是栈上面。

通过优化了内存管理机制，解放广大程序员的双手。让程序员更关注于业务。

注意：Go 在编译阶段确立逃逸，并不是在运行时。

**什么是栈与堆**

面试官：“那你说下什么是栈和堆”

胖虎心：“这个也简单啊”

栈（ stack）是系统自动分配空间的，例如我们定义一个 char a；系统会自动在栈上为其开辟空间。而堆（heap）则是程序员根据需要自己申请的空间，例如 malloc（10）；开辟十个字节的空间。

先看下内存分配图

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfMp4K-164889565844810.png)

栈在内存中是从高地址向下分配的，并且连续的，遵循先进后出原则。系统在分配的时候已经确定好了栈的大小空间。栈上面的空间是自动回收的，所以栈上面的数据的生命周期在函数结束后，就被释放掉了。

堆分配是从低地址向高地址分配的，每次分配的内存大小可能不一致，导致了空间是不连续的，这也产生内存碎片的原因。由于是程序分配，所以效率相对慢些。而堆上的数据只要程序员不释放空间，就一直可以访问到，不过缺点是一旦忘记释放会造成内存泄露。

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfMn4f-164889565844812.png)

**逃逸分析有什么好处**

面试官：“那你说下逃逸分析有什么好处吗”

胖虎：“你是十万个为什么吗？”， 但胖虎还是掏出了自己的看家本领。

就像刚开始提到的，Go 语言中内存的分配不是有程序员自己决定的，而是通过编译阶段确定的分配到何处。这样有什么好处呢？没错机智的你，可能已经猜到了就是为了优化程序，榨干机器性能，让内存能够得到更高的使用效率。

通过逃逸分析，那些不需要分配到堆上的变量直接分配到栈上，堆上的变量少了不但同时减少 GC 的压力，还减轻了内存分配的开销。

**常见的逃逸现象**

面试官点点头，称赞的眼光看着胖虎说：“那你在说说，常见的逃逸现象有哪些吧”

胖虎内心崩溃了：“就我一个人一直在这说，都要渴死了，倒是给我来杯水啊，能不能让我喘口气”。但一想 JD 上面给的薪资还是挺诱惑人的，那就在回答一题。

**func（函数类型）数据类型**

```go
package main


import "fmt"


func main() {
    name := test()
    fmt.Println(name())
}


func test() func() string {
    return func() string {
        return "后端时光"
    }
}
```

执行命令

```go
go build -gcflags="-m -l" eee.go
```

-m：表示内存分析 -l：表示防止内联优化

结果如下，第11行变量 name 逃逸到了堆上

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfMrr9-164889565844814.png)

**interface{} 数据类型**

```go
package main


import "fmt"


func main() {
    name := "Golang"
    fmt.Println(name)
}
```

同理执行逃逸分析，结果如下， name 变量也逃逸到堆上了

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfMxMQ-164889565844816.png)

原因是 go/src/fmt/print.go 文件中 Println 传参数类型 interface{}, 编译器对传入的变量类型未知，所有统一处理分配到了堆上面去了。

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfQMIx-164889565844818.png)

**指针类型**

```go
package main


import "fmt"


func main() {
    name := point()
    fmt.Println(*name)
}


func point() *string {
    name := "指针"
    return &name
}
```

结果如下，第11行变量 name 逃逸到了堆上

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/qfQtLd-164889565844820.png)

还有其他情况出现变量逃逸吗？

“额，这……”，胖虎心想：时间太匆忙了，八股文我就背了这么点啊，其他的还么来得及看呢，要是昨天少玩一把游戏就好了。这可怎么办？

看着胖虎憋的满脸通红，面试官笑呵呵的说，“没事的，时间也不早了，今天先到这吧，你还有什么要问我的吗？”

胖虎：“还有哪些会出现变量逃逸啊”

面试官：“channel 或者栈空间不足逃逸, 也会导致逃逸的情况”

**原文出处**

https://mp.weixin.qq.com/s/JXLGLya8ryCMS3g6loTZHw

## 获取不到锁会一直等待吗？

会。
在 2016 年 Go 1.9 中 Mutex 增加了饥饿模式，让锁变得更公平，不公平的等待时间限制在 1 毫秒，并且修复了一个大 Bug：总是把唤醒的 goroutine 放在等待队列的尾部，会导致出现不公平的等待时间。那什么时候会进入饥饿模式？1 毫秒，一旦等待者等待时间超过这个时间阈值，就可能会进入饥饿模式，优先让等待着先获取到锁。有饥饿模式自然就有正常模式了，这里就不展开了。你只需要记住，Mutex 锁不会容忍一个 goroutine 被落下，永远没有机会获取锁。Mutex 尽可能地让等待较长的 goroutine 更有机会获取到锁。

## 如何实现一个 timeout 的锁？

用 for 循环和 TryLock 实现。先记录开始的时间，用 for 循环判断是否超时，没有超时则反复尝试 TryLock，直到获取成功；如果超时直接返回失败。可这样有一个问题，高频的 CAS 自旋操作，如果失败的太多，会消耗大量的 CPU，我们需要进行优化，将 TryLock 的抢占实现分为两部分，一个是 fast path，另一个是竞争状态下的，后者的 CAS 操作很多，可以考虑减少 slow 方法的频率，例如调用 n 次 fast path 失败后，再调用一次整个 TryLock。我们还可以借鉴 TCP 重试机制进行优化，for 循环中的重试增加休眠时间，每次失败将休眠时间乘以一个系数，直到达到上限，减少自旋带来的性能损耗。

## go 的切片扩容机制

1.18之前都是在1024之前翻倍扩容，之后是1.25倍
1.18之后在256之后，（1.25倍+192），小切片比较多，减少内存分配次数

## 管道是否能二次关闭？

关闭已关闭的通道

会引发panic: close of closed channel

```go
//关闭一个已经关闭的管道
func main() {
    channel := make(chan int, 10)
    close(channel)
    close(channel)
}
/*[Output]: panic: close of closed channel */
```

## 管道关闭是否能读写？

- 往已关闭的channel写入会引发panic；
- 读已关闭的channel能一直读到东西，但是读到的内容根据通道内关闭前是否有元素而不同。

1. 如果chan关闭前，buffer内有元素还未读,会正确读到chan内的值，且返回的第二个bool值（是否读成功）为true。
2. 如果chan关闭前，buffer内有元素已经被读完，chan内无值，接下来所有接收的值都会非阻塞直接成功，返回 channel 元素的零 值，但是第二个bool值一直为false。

## 问等待所有goroutine结束，怎么做？

### 用channel进行同步(该方法需要知道goroutine的数量)

```go
func main() {
    ch := make(chan int, 2)

    go func() {
        for i := 0; i < 10; i++ {
            time.Sleep(1 * time.Second)
            fmt.Println("go routine1", i)
        }
        ch <- 1
    }()
    go func() {
        for i := 0; i < 10; i++ {
            time.Sleep(1 * time.Second)
            fmt.Println("go routine2", i)
        }
        ch <- 2
    }()

    // 等待
    for i := 0; i < 2; i++ {
        <-ch
    }

    fmt.Println("main exist")
}
```

### 用sync.WaitGroup

```go
package main

import "fmt"
import "time"
import "sync"

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            time.Sleep(1 * time.Second)
            fmt.Println(i)
        }(i)
    }

    wg.Wait() // 等待

    fmt.Println("main exist")
}
```
