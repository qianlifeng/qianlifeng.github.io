---
title: Golang内存分配
date: 2018-12-01 22:11:17
draft: true
---

> 本文基于`Golang 1.11`，其中的观点不一定适用于后续版本

> 在进行代码分析的时候，我会删除不重要的代码，只保留主干部分，这样可以使得流程更加简洁

Go 的内存分配器采用了跟 tcmalloc 库相同的实现，是一个带内存池的分配器，底层直接调用操作系统的 mmap 等函数。根据需要分配对象的大小，Go 使用了不同的内存分配策略。

- 对于`<=32K`的内存，为了优化分配效率，采用了多级分配策略。大致的分配流程如下：
  - 首先每个 P 有自己的内存池(`mcache`)，如果`mcache`足够分配此次内存，则直接从`mcache`内存池分配
  - 如果`mcache`不够，则尝试向`mcentral`内存池（全局唯一）申请
  - 如果`mcentral`也不够，则尝试向`mheap`内存池（全局唯一）申请，此时如果`mheap`不够，则会直接调用系统函数进行内存的申请
- 对于`>32K`的内存，直接分配在`mheap`内存池上分配

# 基本概念

在详细讲解上述内存分配流程之前，有几个基本概念需要了解一下。

- 我们常说的堆内存在 Go 中对应其`arena`区域，这块内存区域是对象最终分配的实际物理内存（对应操作系统的逻辑地址空间，此区域内系统按照页进行内存划分，在`x64`位架构上，每页大小为`8K`）
- 为了管理这些实际物理内存与 Go 对象之间的对应关系，Go 引入了`bitmap`内存区，用于表示 arena 区域中哪些地址保存了对象，并且对象中哪些地址包含了指针。
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

# 内存分配流程

### 内存初始化

分配之前需要进行初始化操作，内存相关的初始化是在`schedinit`中完成的。这个方法是`runtime`初始化调度器用的，系统在启动的时候会调用此方法。

```golang
// runtime/proc.go
// 系统启动流程为:
//
//	call osinit
//	call schedinit
//	make & queue new G
//	call runtime·mstart
//
// 在新生成的G上运行 runtime·main.
func schedinit() {
	// 内存相关初始化
	stackinit()
	mallocinit()

	// 命令行参数以及环境参数初始化
	goargs()
	goenvs()

	// 垃圾回收初始化
	gcinit()
}
```

```golang
// runtime/malloc.go
func mallocinit() {

	// Initialize the heap.
	mheap_.init()
	_g_ := getg()
	_g_.m.mcache = allocmcache()

	// 从1.11开始，go移除了最大内存512G的限制（https://github.com/golang/go/issues/10460）
	// 1.11之前go会一次性向操作系统申请保留512G的虚拟内存（并不会真正使用），并将这块虚拟内存放在一个arena中
	// 1.11开始会生成0x7f个(128个）arenahint，每个arenahint下面每次申请`heapArenaBytes`（64位下默认64M，Windows 64位下面默认为4M），直到达到 1 << 40 (128G)的时候切换到下一个hint继续分配
	if sys.PtrSize == 8 && GOARCH != "wasm" {
		for i := 0x7f; i >= 0; i-- {
			var p uintptr
			switch {
			case GOARCH == "arm64":
				p = uintptr(i)<<40 | uintptrMask&(0x0040<<32)
			}
			// 通过fixalloc分配一个新的arenaHint对象，并将其串到`mheap_.arenaHints`上面
			// fixalloc是一个固定大小的内存分配组件，他有一个list列表，列表上挂的都是固定大小的内存块（内存块的大小在初始化fixalloc的时候指定）。
			// 每次分配内存的时候，优先从list列表里面拿，如果list里面的内存块已经分配完了，则直接向操作系统申请一块内存（chunk），然后从里面分配固定大小出去
			// 内存返还的时候直接挂到list列表上面供下次使用。fixalloc申请的内存都不是在堆上的
			// 可以简单的把fixalloc当成一个带有缓存的对象分配器
			hint := (*arenaHint)(mheap_.arenaHintAlloc.alloc())
			hint.addr = p
			hint.next, mheap_.arenaHints = mheap_.arenaHints, hint
		}
	} else {
		// 32位不做分析
	}
}
```

至此，内存初始化完成。比较简单，没有发生任何实际的内存分配。

### 内存分配

所有堆内存的申请都是通过`newobject`完成的，传入对象类型，然后返回对象的地址。

```golang
// runtime/malloc.go
func newobject(typ *_type) unsafe.Pointer {
	return mallocgc(typ.size, typ, true)
}
```

```golang
// runtime/malloc.go
// 实际分配流程就是我们在文章开始提到的，大对象直接在堆上申请，小对象则在mcache和mcentral上面申请
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	//零长度对象，例如空的struct，因为不会发生读写，所以都是返回固定的一个地址
	if size == 0 {
		return unsafe.Pointer(&zerobase)
	}

	//协助GC机制，当前G想分配内存，但是已经分配了很多的时候，需要协助GC完成一部分的对象扫描工作才能继续申请新的内存
	//官方将这个比喻成G的信用额度，每个G都有申请内存的信用额度，申请一次少一次。当信用额度小于0的时候，需要给GC“打工”来挣取信用
	//这个机制用于防止对象分配太快，GC跟不上的情况
	var assistG *g
	if gcBlackenEnabled != 0 {
		assistG = getg()
		if assistG.m.curg != nil {
			assistG = assistG.m.curg
		}
		assistG.gcAssistBytes -= int64(size)
		if assistG.gcAssistBytes < 0 {
			gcAssistAlloc(assistG)
		}
	}

	c := gomcache()
	if size <= maxSmallSize {  //需要申请的对象<=32K的时候
		if noscan && size < maxTinySize { //如果申请的对象不包含指针且申请的内存<16个字节的时候
			// 微小对象申请流程(主要是一些小字符串)

			// 如果剩余空间足够分配此次内存，则直接从tiny块上分配
			off := c.tinyoffset
			if off+size <= maxTinySize && c.tiny != 0 {
				x = unsafe.Pointer(c.tiny + off)
				c.tinyoffset = off + size
				c.local_tinyallocs++
				mp.mallocing = 0
				releasem(mp)
				return x
			}

			// 不够分配的时候则直接从申请一块新的微小内存块
			// 这里的`nextFreeFast`和`nextFree`用于分配内存，会在后面的代码分析到，这里不展开
			span := c.alloc[tinySpanClass]
			v := nextFreeFast(span)
			if v == 0 {
				v, _, shouldhelpgc = c.nextFree(tinySpanClass)
			}
			x = unsafe.Pointer(v)
			(*[2]uint64)(x)[0] = 0
			(*[2]uint64)(x)[1] = 0

			// 如果原来的内存块剩余不多了，或者还没有分配微小内存块，则将本次新分配的内存块设置上去
			if size < c.tinyoffset || c.tiny == 0 {
				c.tiny = uintptr(x)
				c.tinyoffset = size
			}
			size = maxTinySize
		} else {
			//申请的对象>=16字节 且 <= 32KB
			var sizeclass uint8
			if size <= smallSizeMax-8 {
				sizeclass = size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]
			} else {
				sizeclass = size_to_class128[(size-smallSizeMax+largeSizeDiv-1)/largeSizeDiv]
			}
			size = uintptr(class_to_size[sizeclass])
			spc := makeSpanClass(sizeclass, noscan)
			span := c.alloc[spc]
			v := nextFreeFast(span)
			if v == 0 {
				v, span, shouldhelpgc = c.nextFree(spc)
			}
			x = unsafe.Pointer(v)
			if needzero && span.needzero != 0 {
				memclrNoHeapPointers(unsafe.Pointer(v), size)
			}
		}
	} else {
		//大于32KB的对象直接从堆上申请
		var s *mspan
		shouldhelpgc = true
		systemstack(func() {
			s = largeAlloc(size, needzero, noscan)
		})
		s.freeindex = 1
		s.allocCount = 1
		x = unsafe.Pointer(s.base())
		size = s.elemsize
	}

	var scanSize uintptr
	if !noscan {
		// If allocating a defer+arg block, now that we've picked a malloc size
		// large enough to hold everything, cut the "asked for" size down to
		// just the defer header, so that the GC bitmap will record the arg block
		// as containing nothing at all (as if it were unused space at the end of
		// a malloc block caused by size rounding).
		// The defer arg areas are scanned as part of scanstack.
		if typ == deferType {
			dataSize = unsafe.Sizeof(_defer{})
		}
		heapBitsSetType(uintptr(x), size, dataSize, typ)
		if dataSize > typ.size {
			// Array allocation. If there are any
			// pointers, GC has to scan to the last
			// element.
			if typ.ptrdata != 0 {
				scanSize = dataSize - typ.size + typ.ptrdata
			}
		} else {
			scanSize = typ.ptrdata
		}
		c.local_scan += scanSize
	}

	// Ensure that the stores above that initialize x to
	// type-safe memory and set the heap bits occur before
	// the caller can make x observable to the garbage
	// collector. Otherwise, on weakly ordered machines,
	// the garbage collector could follow a pointer to x,
	// but see uninitialized memory or stale heap bits.
	publicationBarrier()

	// Allocate black during GC.
	// All slots hold nil so no scanning is needed.
	// This may be racing with GC so do it atomically if there can be
	// a race marking the bit.
	if gcphase != _GCoff {
		gcmarknewobject(uintptr(x), size, scanSize)
	}

	if raceenabled {
		racemalloc(x, size)
	}

	if msanenabled {
		msanmalloc(x, size)
	}

	mp.mallocing = 0
	releasem(mp)

	if debug.allocfreetrace != 0 {
		tracealloc(x, size, typ)
	}

	if rate := MemProfileRate; rate > 0 {
		if size < uintptr(rate) && int32(size) < c.next_sample {
			c.next_sample -= int32(size)
		} else {
			mp := acquirem()
			profilealloc(mp, x, size)
			releasem(mp)
		}
	}

	if assistG != nil {
		// Account for internal fragmentation in the assist
		// debt now that we know it.
		assistG.gcAssistBytes -= int64(size - dataSize)
	}

	if shouldhelpgc {
		if t := (gcTrigger{kind: gcTriggerHeap}); t.test() {
			gcStart(gcBackgroundMode, t)
		}
	}

	return x
}
```

![](/images/golang-memory/1.png)

# 内存分配流程图

![](/images/golang-memory/2.webp)

# 参考链接

- [Golang 源码探索(三) GC 的实现原理](https://www.cnblogs.com/zkweb/p/7880099.html)
- [探索 Go 内存管理(分配)](https://www.jianshu.com/p/47691d870756)
- [深入 Golang 之内存管理](http://www.opscoder.info/golang_mem_management.html)
- [雨痕 Golang 学习笔记](https://github.com/qyuhen/book)
