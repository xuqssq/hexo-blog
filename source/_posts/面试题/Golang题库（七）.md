---
title: Golang题库（七）
tags:
  - 面试题
  - Go
categories:
  - 面试题
toc: true
toc_number: true
abbrlink: 41935
date: 2023-06-27 20:14:45
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

# Golang题库（七）

## Slice 与 Array, Append()

### **Array**

数组（Array）是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因其长度的不可变动，数组在Go中很少直接使用。把一个大数组传递给函数会消耗很多内存。一般采用数组的切片

几种初始化方式

```
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
arr3 := [3]int{0:3,1:4}
```

### **Slice**

Slice是一种数据结构，描述与Slice变量本身分开存储的Array的连续部分。 Slice不是Array。Slice描述了Array的一部分。

slice底层是一个struct

```
// runtime/slice.go
type slice struct {
    array unsafe.Pointer// 指向数组的指针
    len   int
    cap   int
}
```

创建slice的几种方法

```
// 直接通过make创建，可以指定len、cap
s4 := make([]int, 5, 10)

// 通过数组/slice 切片生成
var data [10]int
s2 := data[2:8]
s3 := s2[1:3]

// append()
s6 = append(s4,6)

// 直接创建
s1 := []int{1, 2}
```

### **append() 底层逻辑**

1. 计算追加后slice的总长度n
2. 如果总长度n大于原cap，则调用growslice func进行扩容（cap最小为n，具体扩容规则见growslice）
3. 对扩容后的slice进行切片，长度为n，获取slice s，用以存储所有的数据
4. 根据不同的数据类型，调用对应的复制方法，将原slice及追加的slice的数据复制到新的slice

### **growslice 计算cap的逻辑**

1. 原cap扩容一倍，即doublecap
2. 如果指定cap大于doublecap则使用cap，否则执行如下
3. 如果原数据长度小于1024，则使用doublecap
4. 否则在原cap的基础上每次扩容1/4，直至不小于cap

## 如何实现一个线程安全的 map?

### **三种方式实现**：

- 加读写锁
- 分片加锁
- sync.Map

加读写锁、分片加锁，这两种方案都比较常用，后者的性能更好，因为它可以降低锁的粒度，提高访问此 map 对象的吞吐。前者并发性能虽然不如后者，
但是加锁的方式更加简单。sync.Map 是 Go 1.9 增加的一个线程安全的 map ，虽然是官方标准，但反而是不常用的，原因是 map 要解决的场景很难

描述，很多时候程序员在做抉择是否该用它，不过在一些特殊场景会使用 sync.Map，

#### 场景一：

只会增长的缓存系统，一个 key 值写入一次而被读很多次；

#### 场景二：

多个 goroutine 为不相交的键读、写和重写键值对。对它的使用场景介绍，来自[官方文档](https://golang.org/pkg/sync/#Map)，这里就不展开了。
加读写锁，扩展 map 来实现线程安全，支持并发读写。使用读写锁 RWMutex，是为了读写性能的考虑。
对 map 对象的操作，无非就是常见的增删改查和遍历。我们可以将查询和遍历看作读操作，增加、修改和
删除看作写操作。示例代码链接：https://github.com/guowei-gong/go-demo/blob/main/mutex/demo.go
。通过读写锁提供线程安全的 map，但是大量并发读写的情况下，锁的竞争会很激烈，导致性能降低。如何解决这个问题？
尽量减少锁的粒度和锁的持有时间，减少锁的粒度，常用方法就是分片 Shard，将一把锁分成几把锁，每个锁控制一个分片。

## ⭐go 的锁是可重入的吗？

**不是可重入锁。**
讨论这个问题前，先解释一下“重入”这个概念。当一个线程获取到锁时，如果没有其他线程拥有这个锁，那么这个线程就会成功获取到这个锁。线程持有这个锁后，其他线程再请求这个锁，其他线程就会进入阻塞等待的状态。

但是如果拥有这个锁的线程再请求这把锁的话，就不会阻塞，而是成功返回，这就是可重入锁。可重入锁也叫做**递归锁**。
为什么 go 的锁不是可重入锁，因为 Mutex 的实现中，没有记录哪个 goroutine 拥有这把锁。换句话说，我们可以通过
扩展来将 go 的锁变为可重入锁，这里就不展开了。下面是一个误用 Mutex 的重入例子：https://github.com/guowei-gong/go-demo/commit/a6fc236853f5cd0efd4e62269cfe60a19de7a96e

## ⭐Go map 的底层实现 ？

Go语言的map使用Hash表和搜索树作为底层实现，一个Hash表可以有多个bucket，而每个bucket保存了map中的一个或一组键值对。

**源码：**`runtime/map.go:hmap`

```go
// go 1.17
type hmap struct {
    count      int            //元素个数，调用len(map)时直接返回
    flags      uint8          //标志map当前状态,正在删除元素、添加元素.....
    B          uint8          //单元(buckets)的对数 B=5表示能容纳32个元素
    noverflow  uint16        //单元(buckets)溢出数量，如果一个单元能存8个key，此时存储了9个，溢出了，就需要再增加一个单元
    hash0      uint32         //哈希种子
    buckets    unsafe.Pointer //指向单元(buckets)数组,大小为2^B，可以为nil
    oldbuckets unsafe.Pointer //扩容的时候，buckets长度会是oldbuckets的两倍
    nevacute   uintptr        //指示扩容进度，小于此buckets迁移完成
    extra      *mapextra      //与gc相关 可选字段
}
```

下图展示了一个拥有4个bucket的map。

![image-20220403085916040](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220403085916040.png)

本例中，hmap.B=2，hmap.buckets数组的长度是4 (2B)。元素经过Hash运算后会落到某个bucket中进行存储。

**bucket的数据结构**

数据结构源码：`runtime/map.go/bmap`

```go
// A bucket for a Go map.
type bmap struct {
	tophash [bucketCnt]uint8
}
//实际上编译期间会生成一个新的数据结构
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

bmp也就是bucket，由初始化的结构体可知，里面最多存8个key，每个key落在桶的位置有hash出来的结果的高8位决定。

其中tophash是一个长度为8的整型数组，Hash值相同的键存入当前bucket时会将Hash值的高位存储在该数组中，以便后续匹配。

整体图如下

![image-20220403092625292](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220403092625292.png)

有一点需要注意：当`map`的`key`和`value`都不是指针，并且`size`都小于 128 字节的情况下，会把 `bmap`标记为不含指针，这样可以避免`gc`时扫描整个`hmap`。尽管如此，但如图所示，`bmap`是有一个`overflow`的字段，该字段是指针类型，这就破坏了`bmap`不含指针的设想，这时会把`overflow`移动到`extra`字段来。
