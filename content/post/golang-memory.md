---
title: Golang内存管理
date: 2018-12-01 22:11:17
tags: [Golang]
draft: true
---

Go 的内存分配器采用了跟 tcmalloc 库相同的实现，是一个带内存池的分配器，底层直接调用操作系统的 mmap 等函数。根据需要分配对象的大小，Go 使用了不同的内存分配策略。

- 对于`<=32K`的内存，为了优化分配效率，采用了多级分配策略。大致的分配流程如下：
  - 首先每个 P 有自己的内存池(`mcache`)，如果`mcache`足够分配此次内存，则直接从`mcache`内存池分配
  - 如果`mcache`不够，则尝试向`mcentral`内存池（全局唯一）申请内存
  - 如果`mcentral`也不够，则尝试向`mheap`内存池（全局唯一）申请，此时如果`mheap`不够，则会直接调用系统函数进行内存的申请
- 对于`>32K`的内存，直接分配在`mheap`内存池上分配

# 基本概念

在详细讲解上述内存分配流程之前，有几个基本概念需要了解一下。

- 我们常说的堆内存实际是对应了 Go 中的`arena`区域，这块内存区域是对象最终分配的实际物理内存（对应操作系统的逻辑地址空间，此区域内系统按照页进行内存划分，在`x64`位架构上，每页大小为`8K`）
- 为了管理这些实际物理内存与 Go 对象之间的对应关系 Go 引入了`bitmap`内存区，用于表示 arena 区域中哪些地址保存了对象，并且对象中哪些地址包含了指针。
- 为了抽象内存分配流程，Go 中引入了`span`的概念，`span`是逻辑上的 Go 中内存分配的最小单位。

![](/images/golang-memory/arena.png)

# mspan

- 管理内存的基本单位
- mspan 负责管理从 startAddr 开始的 N 个 page 的地址空间

```golang
//保留重要成员变量
type mspan struct {
	next *mspan     // 链表中下个span
	prev *mspan     // 链表中上个span
	startAddr uintptr // 该mspan的起始地址
	freeindex uintptr // 表示分配到第几个块
	npages    uintptr // 一个span中含有几页
	sweepgen    uint32 // GC相关
	incache     bool       // 是否被mcache占用
	spanclass   spanClass  // 0 ~ _NumSizeClasses之间的一个值，比如，为3，那么这个mspan被分割成32byte的块
}
```

# mcache

- 每个 P 一个 mcache，用来存储小内存(<=32KB)
- mcache 上面的内存不会被 GC 到
- 每个 mcache 下面分了 67 个 mspan，每个 mspan 用来存储固定大小的内存

```golang
type mcache struct {
	tiny             uintptr // 小对象分配器
	tinyoffset       uintptr // 小对象分配偏移
	local_tinyallocs uintptr // number of tiny allocs not counted in other stats
	alloc [numSpanClasses]*mspan // 存储不同级别的mspan
}
```

# mcentral

- 如果需要分配的内存>32KB，则通过 mcentral 分配（其内部也是通过 mspan 实际存储，mcentral 只是逻辑上对 mspani 进行管理）
- 每个 mcentral 都有自己存放内存的固定大小
- 每个 mcentral 有里两个 mspan 列表，nonempty 对应待分配的，empty 对应已经被分配的
- 从 mcentral 申请内存的时候需要加锁

```golang
type mcentral struct {
	lock      mutex    // 多个P会访问，需要加锁
	spanclass spanClass  // 对应了mspan中的spanclass
	nonempty  mSpanList // 该mcentral可用的mspan列表
	empty     mSpanList // 该mcentral中已经被使用的mspan列表
}
```

# mheap

- arena_start 向高地址方向增长的是内存池，向低地址方向增长的是 bitmap。
- arena 是 Golang 中用于分配内存的连续虚拟地址区域。堆上申请的所有内存都来自 arena。操作系统常见有两种做法标志内存可用：一种是用链表将所有的可用内存都串起来；另一种是使用位图来标志内存块是否可用。

```golang
type mheap struct {
	lock      mutex                    // 是公有的，需要加锁
	free      [_MaxMHeapList]mSpanList // 未分配的spanlist，比如free[3]是由包含3个 page 的 mspan 组成的链表
	freelarge mTreap                   // mspan组成的链表，每个mspan的 page 个数大于_MaxMHeapList
	busy      [_MaxMHeapList]mSpanList // busy lists of large spans of given length
	busylarge mSpanList                // busy lists of large spans length >= _MaxMHeapList
	allspans []*mspan                  // 所有申请过的 mspan 都会记录在 allspans
	spans []*mspan                     // 记录 arena 区域页号（page number）和 mspan 的映射关系

	arena_start uintptr // arena是Golang中用于分配内存的连续虚拟地址区域，这是该区域开始的指针
	arena_used  uintptr // 已经使用的内存的指针
	arena_alloc uintptr
	arena_end   uintptr

	central [numSpanClasses]struct {
		mcentral mcentral
		pad      [sys.CacheLineSize - unsafe.Sizeof(mcentral{})%sys.CacheLineSize]byte //避免伪共享（false sharing）问题
	}

	spanalloc             fixalloc // allocator for span*
	cachealloc            fixalloc // mcache分配器

}
```

![](/images/golang-memory/1.png)

# 内存分配流程图

![](/images/golang-memory/2.webp)

# 参考链接

- [Golang 源码探索(三) GC 的实现原理](https://www.cnblogs.com/zkweb/p/7880099.html)
- [探索 Go 内存管理(分配)](https://www.jianshu.com/p/47691d870756)
- [深入 Golang 之内存管理](http://www.opscoder.info/golang_mem_management.html)
