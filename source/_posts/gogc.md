---
title: GO垃圾回收机制
date: 2021-02-19 14:32:47
tags:
 - GC
 - 垃圾回收
 - golang
categories:
 - GO
---

在此之前，我们介绍过 [浅析Python的垃圾回收机制](https://runnerliu.github.io/2017/07/16/pythongc/) 和 [PHP新的垃圾回收机制](https://runnerliu.github.io/2017/04/15/phpnewgc/) ，有兴趣的话可以参考阅读，今天我们来聊聊golang是如何进行垃圾回收的。我们知道，目前各语言进行垃圾回收的方法有很多，如引用计数、标记清除、分代回收、三色标记等，各种方式都有其特点，GO语言在发展过程中， 其GC算法也是不断改进的。

### GO的GC里程碑

#### v1.3以前：STW

golang的垃圾回收算法都非常简陋，其性能也广被诟病：go runtime在一定条件下（内存超过阈值或定期如2min），暂停所有任务的执行，进行mark&sweep操作，操作完成后启动所有任务的执行。在内存使用较多的场景下，go程序在进行垃圾回收时会发生非常明显的卡顿现象（Stop The World）。在对响应速度要求较高的后台服务进程中，这种延迟简直是不能忍受的！这个时期国内外很多在生产环境实践go语言的团队都或多或少踩过gc的坑。当时解决这个问题比较常用的方法是尽快控制自动分配内存的内存数量以减少gc负荷，同时采用手动管理内存的方法处理需要大量及高频分配内存的场景。

#### v1.3：Mark STW & Sweep

1.3版本中，go runtime分离了mark和sweep操作，和以前一样，也是先暂停所有任务执行并启动mark，mark完成后马上就重新启动被暂停的任务了，而是让sweep任务和普通协程任务一样并行的和其他任务一起执行。如果运行在多核处理器上，go会试图将gc任务放到单独的核心上运行而尽量不影响业务代码的执行。go team自己的说法是减少了50%-70%的暂停时间。

#### v1.5：三色标记

go 1.5正在实现的垃圾回收器“非分代的、非移动的、并发的、三色的标记清除垃圾收集器”。这种方法的mark操作是可以渐进执行的而不需每次都扫描整个内存空间，可以减少stop the world的时间。 由此可以看到，一路走来直到1.5版本，go的垃圾回收性能也是一直在提升。

#### v1.8：混合写屏障（hybrid write barrier）

由于标记操作和用户逻辑是并发执行的，用户逻辑会时常生成对象或者改变对象的引用。例如把⼀个对象标记为白色准备回收时，用户逻辑突然引用了它，或者又创建了新的对象。由于对象初始时都看为白色，会被 GC 回收掉，为了解决这个问题，引入了写屏障机制。

GC 对扫描过后的对象使⽤操作系统写屏障功能来监控这段内存。如果这段内存发⽣引⽤改变，写屏障会给垃圾回收期发送⼀个信号，垃圾回收器捕获到信号后就知道这个对象发⽣改变，然后重新扫描这个对象，看看它的引⽤或者被引⽤是否改变。利⽤状态的重置实现当对象状态发⽣改变的时候，依然可以再次其引用的对象。

### GO的GC

#### 三色标记

传统的标记清除算法中，垃圾收集器从垃圾收集的根对象出发，递归遍历这些对象指向的子对象并将所有可达的对象标记成存活；标记阶段结束后，垃圾收集器会依次遍历堆中的对象并清除其中的垃圾，整个过程需要标记对象的存活状态，用户程序在垃圾收集的过程中也不能执行，我们需要用到更复杂的机制来解决 STW 的问题，这就出现了三色标记法。

三色标记算法将程序中的对象分成白色、黑色和灰色三类：

- 白色对象：潜在的垃圾，其内存可能会被垃圾收集器回收；
- 黑色对象：活跃的对象，包括不存在任何引用外部指针的对象以及从根对象可达的对象；
- 灰色对象：活跃的对象，因为存在指向白色对象的外部指针，垃圾收集器会扫描这些对象的子对象；

![2021-2-19T154834.png](/images/2021-2-19T154834.png)

在垃圾收集器开始工作时，程序中不存在任何的黑色对象，垃圾收集的根对象会被标记成灰色，垃圾收集器只会从灰色对象集合中取出对象开始扫描，当灰色集合中不存在任何对象时，标记阶段就会结束。

![2021-2-19T154948.png](/images/2021-2-19T154948.png)

三色标记垃圾收集器的工作原理很简单，我们可以将其归纳成以下几个步骤：

- 从灰色对象的集合中选择一个灰色对象并将其标记成黑色；
- 将黑色对象指向的所有对象都标记成灰色，保证该对象和被该对象引用的对象都不会被回收；
- 重复上述两个步骤直到对象图中不存在灰色对象。

当三色的标记清除的标记阶段结束之后，应用程序的堆中就不存在任何的灰色对象，我们只能看到黑色的存活对象以及白色的垃圾对象，垃圾收集器可以回收这些白色的垃圾，下面是使用三色标记垃圾收集器执行标记后的堆内存，堆中只有对象 D 为待回收的垃圾：

![2021-2-19T155305.png](/images/2021-2-19T155305.png)

因为用户程序可能在标记执行的过程中修改对象的指针，所以三色标记清除算法本身是不可以并发或者增量执行的，它仍然需要 STW，在如下所示的三色标记过程中，用户程序建立了从 A 对象到 D 对象的引用，但是因为程序中已经不存在灰色对象了，所以 D 对象会被垃圾收集器错误地回收。

![2021-2-19T155403.png](/images/2021-2-19T155403.png)

本来不应该被回收的对象却被回收了，这在内存管理中是非常严重的错误，我们将这种错误称为悬挂指针，即指针没有指向特定类型的合法对象，影响了内存的安全性，想要并发或者增量地标记对象还是需要使用屏障技术。

整个流程如下：

![2021-2-19T155548.gif](/images/2021-2-19T155548.gif)

#### 混合写屏障

想要在并发或者增量的标记算法中保证正确性，我们需要达成以下两种三色不变性（Tri-color invariant）中的一种：

- 强三色不变性：黑色对象不会指向白色对象，只会指向灰色对象或者黑色对象；
- 弱三色不变性：黑色对象指向的白色对象必须包含一条从灰色对象经由多个白色对象的可达路径。

![2021-2-19T160000.png](/images/2021-2-19T160000.png)

上图分别展示了遵循强三色不变性和弱三色不变性的堆内存，遵循上述两个不变性中的任意一个，我们都能保证垃圾收集算法的正确性，而屏障技术就是在并发或者增量标记过程中保证三色不变性的重要技术。

垃圾收集中的屏障技术更像是一个钩子方法，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，根据操作类型的不同，我们可以将它们分成读屏障（Read barrier）和写屏障（Write barrier）两种，因为读屏障需要在读操作中加入代码片段，对用户程序的性能影响很大，所以编程语言往往都会采用写屏障保证三色不变性。

Go 语言在 v1.8 组合 Dijkstra 插入写屏障和 Yuasa 删除写屏障构成混合写屏障，该写屏障会**将被覆盖的对象标记成灰色并在当前栈没有扫描时将新对象也标记成灰色**。

为了移除栈的重扫描过程，除了引入混合写屏障之外，在垃圾收集的标记阶段，我们还需要**将创建的所有新对象都标记成黑色**，防止新分配的栈内存和堆内存中的对象被错误地回收，因为栈内存在标记阶段最终都会变为黑色，所以不再需要重新扫描栈空间。

#### 增量和并发

传统的垃圾收集算法会在垃圾收集的执行期间暂停应用程序，一旦触发垃圾收集，垃圾收集器会抢占 CPU 的使用权占据大量的计算资源以完成标记和清除工作，然而很多追求实时的应用程序无法接受长时间的 STW。

为了减少应用程序暂停的最长时间和垃圾收集的总暂停时间，我们会使用下面的策略优化现代的垃圾收集器：

- 增量垃圾收集：增量地标记和清除垃圾，降低应用程序暂停的最长时间；
- 并发垃圾收集：利用多核的计算资源，在用户程序执行时并发标记和清除垃圾；

因为增量和并发两种方式都可以与用户程序交替运行，所以我们需要使用屏障技术保证垃圾收集的正确性；与此同时，应用程序也不能等到内存溢出时触发垃圾收集，因为当内存不足时，应用程序已经无法分配内存，这与直接暂停程序没有什么区别，增量和并发的垃圾收集需要提前触发并在内存不足前完成整个循环，避免程序的长时间暂停。

##### 增量收集

增量式（Incremental）的垃圾收集是减少程序最长暂停时间的一种方案，它可以将原本时间较长的暂停时间切分成多个更小的 GC 时间片，虽然从垃圾收集开始到结束的时间更长了，但是这也减少了应用程序暂停的最大时间：

![2021-2-19T160403.png](/images/2021-2-19T160403.png)

需要注意的是，增量式的垃圾收集需要与三色标记法一起使用，为了保证垃圾收集的正确性，我们需要在垃圾收集开始前打开写屏障，这样用户程序修改内存都会先经过写屏障的处理，保证了堆内存中对象关系的强三色不变性或者弱三色不变性。虽然增量式的垃圾收集能够减少最大的程序暂停时间，但是增量式收集也会增加一次 GC 循环的总时间，在垃圾收集期间，因为写屏障的影响用户程序也需要承担额外的计算开销，所以增量式的垃圾收集也不是只带来好处的，但是总体来说还是利大于弊。

##### 并发收集

并发（Concurrent）的垃圾收集不仅能够减少程序的最长暂停时间，还能减少整个垃圾收集阶段的时间，通过开启读写屏障、利用多核优势与用户程序并行执行，并发垃圾收集器确实能够减少垃圾收集对应用程序的影响：

![2021-2-19T160457.png](/images/2021-2-19T160457.png)

虽然并发收集器能够与用户程序一起运行，但是并不是所有阶段都可以与用户程序一起运行，部分阶段还是需要暂停用户程序的，不过与传统的算法相比，并发的垃圾收集可以将能够并发执行的工作尽量并发执行；当然，因为读写屏障的引入，并发的垃圾收集器也一定会带来额外开销，不仅会增加垃圾收集的总时间，还会影响用户程序，这是我们在设计垃圾收集策略时必须要注意的。

### GC的时机

运行时会通过如下所示的 [`runtime.gcTrigger.test`](https://draveness.me/golang/tree/runtime.gcTrigger.test) 方法决定是否需要触发垃圾收集，当满足触发垃圾收集的基本条件时 — 允许垃圾收集、程序没有崩溃并且没有处于垃圾收集循环，该方法会根据三种不同方式触发进行不同的检查：

```
func (t gcTrigger) test() bool {
	if !memstats.enablegc || panicking != 0 || gcphase != _GCoff {
		return false
	}
	switch t.kind {
	case gcTriggerHeap:
		return memstats.heap_live >= memstats.gc_trigger
	case gcTriggerTime:
		if gcpercent < 0 {
			return false
		}
		lastgc := int64(atomic.Load64(&memstats.last_gc_nanotime))
		return lastgc != 0 && t.now-lastgc > forcegcperiod
	case gcTriggerCycle:
		return int32(t.n-work.cycles) > 0
	}
	return true
}
```

1. `gcTriggerHeap` ：堆内存的分配达到达控制器计算的触发堆大小；
2. `gcTriggerTime` ：如果一定时间内没有触发，就会触发新的循环，该出发条件由 [`runtime.forcegcperiod`](https://draveness.me/golang/tree/runtime.forcegcperiod) 变量控制，默认为 2 分钟；
3. `gcTriggerCycle`：如果当前没有开启垃圾收集，则触发新的循环；
4. [`runtime.gcpercent`](https://draveness.me/golang/tree/runtime.gcpercent) 是触发垃圾收集的内存增长百分比，默认情况下为 100，即堆内存相比上次垃圾收集增长 100% 时应该触发 GC，并行的垃圾收集器会在到达该目标前完成垃圾收集。

用于开启垃圾收集的方法 [`runtime.gcStart`](https://draveness.me/golang/tree/runtime.gcStart) 会接收一个 [`runtime.gcTrigger`](https://draveness.me/golang/tree/runtime.gcTrigger) 类型的结构，所有出现 [`runtime.gcTrigger`](https://draveness.me/golang/tree/runtime.gcTrigger) 结构体的位置都是触发垃圾收集的代码：

- [`runtime.sysmon`](https://draveness.me/golang/tree/runtime.sysmon) 和 [`runtime.forcegchelper`](https://draveness.me/golang/tree/runtime.forcegchelper) ：后台运行定时检查和垃圾收集；
- [`runtime.GC`](https://draveness.me/golang/tree/runtime.GC) ：用户程序手动触发垃圾收集；
- [`runtime.mallocgc`](https://draveness.me/golang/tree/runtime.mallocgc) ：申请内存时根据堆大小触发垃圾收集。

![2021-2-19T162340.png](/images/2021-2-19T162340.png)



Read More:

> [垃圾收集器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)
>
> [GO GC 垃圾回收机制](https://segmentfault.com/a/1190000018161588)
>
> [搞懂Go垃圾回收](https://juejin.cn/post/6844903917650722829)
>
> [Go 语言 GC 机制 · Analyze](https://wingsxdu.com/post/golang/gc/)