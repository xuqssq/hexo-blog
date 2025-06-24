---
title: Golang题库（四）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 5389
date: 2023-06-23 20:55:21
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

# Golang题库（四）

## map取一个key，然后修改这个值，原map数据的值会不会变化？

map属于引用类型，所以取一个key，然后修改这个值，原map数据的值会发生变化

## 向为nil的channel发送数据会怎么样

**空通道即无缓冲通道**。无缓冲通道上的发送操作将会阻塞，直到另一个goroutine在对应的通道上执行接收操作，这时值传送完成，两个goroutine都可以继续执行。相反，如果接收操作先执行，接收方gorountine将阻塞，直到另一个goroutine在同一个通道上发送一个值。

使用无缓冲通道进行的通信导致发送和接收goroutine同步化。因此，无缓冲通道也称为*同步通道*。当一个值在无缓冲通道上传递时，接收值后发送方goroutine才被再次唤醒。

## WaitGroup的坑

1. Add一个负数
如果计数器的值小于0会直接panic

2. Add在Wait之后调用
比如一些子协程开头调用Add结束调用Wait，这些 Wait无法阻塞子协程。正确做法是在开启子协程之前先Add特定的值。

3. 未置为0就重用
WaitGroup可以完成一次编排任务，计数值降为0后可以继续被其他任务所用，但是不要在还没使用完的时候就用于其他任务，这样由于带着计数值，很可能出问题。

4. 复制waitgroup
WaitGroup有nocopy字段，不能被复制。也意味着WaitGroup不能作为函数的参数。

## go struct 能不能比较

需要具体情况具体分析，如果struct中含有不能被比较的字段类型，就不能被比较，如果struct中所有的字段类型都支持比较，那么就可以被比较。

不可被比较的类型:
① slice，因为slice是引用类型，除非是和nil比较
② map，和slice同理，如果要比较两个map只能通过循环遍历实现
③ 函数类型

其他的类型都可以比较。

还有两点值得注意：

- 结构体之间只能比较它们是否相等，而不能比较它们的大小。
- 只有所有属性都相等而属性顺序都一致的结构体才能进行比较。
