---
title: Golang题库（五）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 55219
date: 2023-06-25 21:46:12
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

# Golang题库（五）

## go 实现不重启热部署

### 根据SIGHUP 信号量

根据系统的 SIGHUP 信号量，以此信号量触发进程重启，达到热更新的效果。

热部署我们需要考虑几个能力：

- 新进程启动成功，老进程不会有资源残留
- 新进程初始化的过程中，服务不会中断
- 新进程初始化失败，老进程仍然继续工作
- 同一时间，只能有一个更新动作执行

监听信号量的方法的环境是在 类 UNIX 系统中，在现在的 UNIX 内核中，允许多个进程同时监听一个端口。在收到 SIGHUP 信号量时，先 fork 出一个新的进程监听端口，同时等待旧进程处理完已经进来的连接，最后杀掉旧进程。

### 使用air包

`air`包可以实现插件化热更新的方案，集成方便，非常适用于小型项目的开发。



### ⭐saas灰度发布

#### 介绍

灰度发布常见有三种模式`金丝雀发布`、`滚动发布`、`蓝绿发布`。灰度发布主要是更新服务中，通过服务的`多节点、多切片`进行无感知的服务更新迭代。阿里云、AWS等云平台都已支持灰度发布功能。

#### 实现

灰度发布主要是通过网关转发的均衡负载，确保服务更新过程中能够`不停机`、`无感知`、`可回溯`。常见的灰度发布实现方案有`Nginx +Lua + Redis 实现灰度发布`、`Openresty+Lua+Redis灰度发布`、`Treafit+golang+Redis灰度发布`。

#### 缺点

一套成熟的灰度发布系统，是需要有运维去维护的，需要一定的人力成本。同时灰度发布不适用于`敏捷开发`，对于需要敏捷开发的团队来说，通常业务变动频繁，版本管理和代码审核都会缺少严谨，徒增成本。

## 🪫（待补充）读写锁底层是怎么实现的

读写锁的底层是基于互斥锁实现的。

- 为什么有读写锁，它解决了什么问题？（使用场景）
- 它的底层原理是什么？

在这里我会结合 Go 中的读写锁 RWMutex 进行介绍。

我们通过与 Mutex 对比得出答案。Mutex 是不区分 goroutine 对共享资源的操作行为的，在读操作、它会上锁，在写操作，它也会上锁，当一段时间内，读操作居多时，读操作在 Mutex 的保护下也不得不变为串行访问，对性能的影响也就比较大了。

RWMutex 读写锁的诞生为了区分读写操作，在进行读操作时，goroutine 就不必傻傻的等待了，而是可以并发地访问共享资源，将串行读变成了并行读，提高了读操作的性能。

读写锁针对解决一类问题：readers-writes ，同时有多个读或者多个写操作时，只要有一个线程在执行写操作，其他的线程都不能进行读操作。

读写锁其实有三种工作模型：

- Read-perferring优先读设计，可能会导致写饥饿
- Write-prferring优先写设计，避免写饥饿
- 不指定优先级不区分优先级，解决饥饿问题

Go 中的读写锁，工作模型是 Write-prferring 方案。





1. 读写锁解决问题

   主要应用于写操作少，读操作多的场景。读写锁满足以下四条规则。

   - 写锁需要阻塞写锁：一个协程拥有写锁时，其他协程写锁定需要阻塞；
   - 写锁需要阻塞读锁：一个协程拥有写锁时，其他协程读锁定需要阻塞；
   - 读锁需要阻塞写锁：一个协程拥有读锁时，其他协程写锁定需要阻塞；
   - 读锁不能阻塞读锁：一个协程拥有读锁时，其他协程也可以拥有读锁。

2. 读写锁底层实现

   读写锁内部仍有一个互斥锁，用于将多个写操作隔离开来，其他几个都用于隔离读操作和写操作。

   源码包`src/sync/rmmutex.go:RWMutex`中定义了读写锁的数据结构 

   ```go
   type RWMutex struct {
       w Mutex // held if there are pending writers
       writerSem uint32 // semaphore for writers to wait for completing readers
       readerSem uint32 // semaphore for readers to wait for completing writers
       readerCount int32 // number of pending readers
       readerWait int32 // number of departing readers
   }
   ```

   

## 数组是如何实现用下标访问任意元素的

数组是分配了一段`连续内存`的类型，数组索引函数通过数组类型记录的`第一个元素地址`即数组起始地址开始进行运算，下标从`0`开始，每`+1`即从起始地址 `+1`进行取值。

## 2个协程交替打印字母和数字

```go
package main

import (
	"fmt"
)

func main() {
	limit := 26

	numChan := make(chan int, 1)
	charChan := make(chan int, 1)
	mainChan := make(chan int, 1)
	charChan <- 1

	go func() {
		for i := 0; i < limit; i++ {
			<-charChan
			fmt.Printf("%c\n", 'a'+i)
			numChan <- 1

		}
	}()
	go func() {
		for i := 0; i < limit; i++ {
			<-numChan
			fmt.Println(i)
			charChan <- 1

		}
		mainChan <- 1
	}()
	<-mainChan
	close(charChan)
	close(numChan)
	close(mainChan)
}
```

## 🪫（待补充，python协程线程进程机制更复杂）goroutine与线程的区别？

- 一个线程可以有多个协程
- 线程、进程都是同步机制，而协程是异步
- 协程可以保留上一次调用时的状态，当过程重入时，相当于进入了上一次的调用状态
- 协程是需要线程来承载运行的，所以协程并不能取代线程，线程是被分割的CPU资源，协程是组织好的代码流程

