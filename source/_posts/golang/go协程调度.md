---
title: go协程调度
date: 2018-09-19 10:34:08
tags:
  - go
categories:
  - go
---

## go 协程调度

### G-M 模型

缺点

- 单一全局互斥锁(Sched.Lock)和集中状态存储的存在导致所有 goroutine 相关操作，比如：创建、重新调度等都要上锁；
- goroutine 传递问题：M 经常在 M 之间传递『可运行』的 goroutine，这导致调度延迟增大以及额外的性能损耗；
- 每个 M 做内存缓存，导致内存占用过高，数据局部性较差；
- 由于 syscall 调用而形成的剧烈的 worker thread 阻塞和解除阻塞，导致额外的性能损耗。

### G-P-M 模型调度

Go 调度器工作时会维护两种用来保存 G 的任务队列：一种是一个 Global 任务队列，一种是每个 P 维护的 Local 任务队列。

当通过 go 关键字创建一个新的 goroutine 的时候，它会优先被放入 P 的本地队列。为了运行 goroutine，M 需要持有（绑定）一个 P，接着 M 会启动一个 OS 线程，循环从 P 的本地队列里取出一个 goroutine 并执行。当然还有上文提及的 work-stealing 调度算法：当 M 执行完了当前 P 的 Local 队列里的所有 G 后，P 也不会就这么在那躺尸啥都不干，它会先尝试从 Global 队列寻找 G 来执行，如果 Global 队列为空，它会随机挑选另外一个 P，从它的队列里中拿走一半的 G 到自己的队列中执行。

如果一切正常，调度器会以上述的那种方式顺畅地运行，但这个世界没这么美好，总有意外发生，以下分析 goroutine 在两种例外情况下的行为。

Go runtime 会在下面的 goroutine 被阻塞的情况下运行另外一个 goroutine：

- blocking syscall (for example opening a file)
- network input
- channel operations
- primitives in the sync package
  这四种场景又可归类为两种类型：

#### 用户态阻塞/唤醒

当 goroutine 因为 channel 操作或者 network I/O 而阻塞时（实际上 golang 已经用 netpoller 实现了 goroutine 网络 I/O 阻塞不会导致 M 被阻塞，仅阻塞 G，这里仅仅是举个栗子），对应的 G 会被放置到某个 wait 队列(如 channel 的 waitq)，该 G 的状态由\_Gruning 变为\_Gwaitting，而 M 会跳过该 G 尝试获取并执行下一个 G，如果此时没有 runnable 的 G 供 M 运行，那么 M 将解绑 P，并进入 sleep 状态；当阻塞的 G 被另一端的 G2 唤醒时（比如 channel 的可读/写通知），G 被标记为 runnable，尝试加入 G2 所在 P 的 runnext，然后再是 P 的 Local 队列和 Global 队列。

#### 系统调用阻塞

当 G 被阻塞在某个系统调用上时，此时 G 会阻塞在\_Gsyscall 状态，M 也处于 block on syscall 状态，此时的 M 可被抢占调度（可以抢占其他 M）：执行该 G 的 M 会与 P 解绑，而 P 则尝试与其它 idle 的 M 绑定，继续执行其它 G。如果没有其它 idle 的 M，但 P 的 Local 队列中仍然有 G 需要执行，则创建一个新的 M；当系统调用完成后，G 会重新尝试获取一个 idle 的 P 进入它的 Local 队列恢复执行，如果没有 idle 的 P，G 会被标记为 runnable 加入到 Global 队列。（全局队列用武之地）

以上就是从宏观的角度对 Goroutine 和它的调度器进行的一些概要性的介绍，当然，Go 的调度中更复杂的抢占式调度、阻塞调度的更多细节，大家可以自行去找相关资料深入理解，本文只讲到 Go 调度器的基本调度过程，为后面自己实现一个 Goroutine Pool 提供理论基础，这里便不再继续深入上述说的那几个调度了，事实上如果要完全讲清楚 Go 调度器，一篇文章的篇幅也实在是捉襟见肘，所以想了解更多细节的同学可以去看看 Go 调度器 G-P-M 模型的设计者 Dmitry Vyukov 写的该模型的设计文档《Go Preemptive Scheduler Design》以及直接去看源码，G-P-M 模型的定义放在 src/runtime/runtime2.go 里面，而调度过程则放在了 src/runtime/proc.go 里。

## 问题

go 协程阻塞时如何进行调度？

> 在程序中任何对系统 API 的调用，都会被 runtime 层拦截来方便调度。
> Goroutine 在 system call 和 channel call 时都可能发生阻塞，但这两种阻塞发生后，处理方式又不一样的。 1.当程序发生 system call，M 会发生阻塞，同时唤起（或创建）一个新的 M 继续执行其他的 G。当 MO 返回时，它必须尝试取得一个 context P 来运行 goroutine，一般情况下，它会从其他的 OS 线程那里 steal 偷一个 context 过来，如果没有偷到的话，它就把 goroutine 放在一个 global runqueue 里，然后自己就去睡大觉了（放入线程缓存里）。Contexts 们也会周期性的检查 global runqueue，否则 global runqueue 上的 goroutine 永远无法执行。 2.当程序发起一个 channel call，程序可能会阻塞，但不会阻塞 M，G 的状态会设置为 waiting，M 继续执行其他的 G。当 G 的调用完成，会有一个可用的 M 继续执行它。

go 为什么要实现自己的协程调度，而不用系统调度？

> 1.线程较多时，开销较大。
> 2.OS 的调度，程序不可控。而 Go GC 需要停止所有的线程，使内存达到一致状态。

Goroutine vs OS thread 有什么区别？

> 其实 goroutine 用到的就是线程池的技术，当 goroutine 需要执行时，会从 thread pool 中选出一个可用的 M 或者新建一个 M。而 thread pool 中如何选取线程，扩建线程，回收线程，Go Scheduler 进行了封装，对程序透明，只管调用就行，从而简化了 thread pool 的使用。

sysmon 功能是什么？

> 释放闲置超过 5 分钟的 span 物理内存；
> 如果超过 2 分钟没有垃圾回收，强制执行；
> 将长时间未处理的 netpoll 结果添加到任务队列；
> 向长时间运行的 G 任务发出抢占调度；
> 收回因 syscall 长时间阻塞的 P；
