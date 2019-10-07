---
title: go协程调度
date: 2018-09-19 10:34:08
tags:
- go
categories: 
- go
---

### go协程调度

### 核心图
![](https://ws1.sinaimg.cn/large/e5320b2aly1fvenffzk2mj20lu0izq74.jpg)

![](https://ws1.sinaimg.cn/large/e5320b2aly1fvenfxq3kpj20io0ao0w1.jpg)

![](https://ws1.sinaimg.cn/large/e5320b2aly1fv6vtv5mjxj20fa0b4myk.jpg)

![](https://ws1.sinaimg.cn/large/e5320b2aly1fv6vuds7fjj20zk0mxdgx.jpg)
### G-P-M 模型调度
Go调度器工作时会维护两种用来保存G的任务队列：一种是一个Global任务队列，一种是每个P维护的Local任务队列。

当通过go关键字创建一个新的goroutine的时候，它会优先被放入P的本地队列。为了运行goroutine，M需要持有（绑定）一个P，接着M会启动一个OS线程，循环从P的本地队列里取出一个goroutine并执行。当然还有上文提及的 work-stealing调度算法：当M执行完了当前P的Local队列里的所有G后，P也不会就这么在那躺尸啥都不干，它会先尝试从Global队列寻找G来执行，如果Global队列为空，它会随机挑选另外一个P，从它的队列里中拿走一半的G到自己的队列中执行。

如果一切正常，调度器会以上述的那种方式顺畅地运行，但这个世界没这么美好，总有意外发生，以下分析goroutine在两种例外情况下的行为。

Go runtime会在下面的goroutine被阻塞的情况下运行另外一个goroutine：

- blocking syscall (for example opening a file)
- network input
- channel operations
- primitives in the sync package
这四种场景又可归类为两种类型：

### 用户态阻塞/唤醒
当goroutine因为channel操作或者network I/O而阻塞时（实际上golang已经用netpoller实现了goroutine网络I/O阻塞不会导致M被阻塞，仅阻塞G，这里仅仅是举个栗子），对应的G会被放置到某个wait队列(如channel的waitq)，该G的状态由_Gruning变为_Gwaitting，而M会跳过该G尝试获取并执行下一个G，如果此时没有runnable的G供M运行，那么M将解绑P，并进入sleep状态；当阻塞的G被另一端的G2唤醒时（比如channel的可读/写通知），G被标记为runnable，尝试加入G2所在P的runnext，然后再是P的Local队列和Global队列。

### 系统调用阻塞
当G被阻塞在某个系统调用上时，此时G会阻塞在_Gsyscall状态，M也处于 block on syscall 状态，此时的M可被抢占调度（可以抢占其他M）：执行该G的M会与P解绑，而P则尝试与其它idle的M绑定，继续执行其它G。如果没有其它idle的M，但P的Local队列中仍然有G需要执行，则创建一个新的M；当系统调用完成后，G会重新尝试获取一个idle的P进入它的Local队列恢复执行，如果没有idle的P，G会被标记为runnable加入到Global队列。（全局队列用武之地）

以上就是从宏观的角度对Goroutine和它的调度器进行的一些概要性的介绍，当然，Go的调度中更复杂的抢占式调度、阻塞调度的更多细节，大家可以自行去找相关资料深入理解，本文只讲到Go调度器的基本调度过程，为后面自己实现一个Goroutine Pool提供理论基础，这里便不再继续深入上述说的那几个调度了，事实上如果要完全讲清楚Go调度器，一篇文章的篇幅也实在是捉襟见肘，所以想了解更多细节的同学可以去看看Go调度器 G-P-M 模型的设计者 Dmitry Vyukov 写的该模型的设计文档《Go Preemptive Scheduler Design》以及直接去看源码，G-P-M模型的定义放在src/runtime/runtime2.go里面，而调度过程则放在了src/runtime/proc.go里。


### 问题？
#### 0.go协程阻塞时如何进行调度？
> 在程序中任何对系统 API 的调用，都会被 runtime 层拦截来方便调度。
> Goroutine 在 system call 和 channel call 时都可能发生阻塞，但这两种阻塞发生后，处理方式又不一样的。
> 1.当程序发生 system call，M 会发生阻塞，同时唤起（或创建）一个新的 M 继续执行其他的 G。当MO返回时，它必须尝试取得一个context P来运行goroutine，一般情况下，它会从其他的OS线程那里steal偷一个context过来，如果没有偷到的话，它就把goroutine放在一个global runqueue里，然后自己就去睡大觉了（放入线程缓存里）。Contexts们也会周期性的检查global runqueue，否则global runqueue上的goroutine永远无法执行。
> 2.当程序发起一个 channel call，程序可能会阻塞，但不会阻塞 M，G 的状态会设置为 waiting，M 继续执行其他的 G。当 G 的调用完成，会有一个可用的 M 继续执行它。




#### 1.go为什么要实现自己的协程调度，而不用系统调度？
> 1.线程较多时，开销较大。
> 2.OS 的调度，程序不可控。而 Go GC 需要停止所有的线程，使内存达到一致状态。

#### 2.GM为啥不行？P有什么作用？
> 1.每个 P 都有一个队列，用来存正在执行的 G。避免 Global Sched Lock。
> 2.每个 M 运行都需要一个 MCache 结构。M Pool 中通常有较多 M，但执行的只有几个，为每个池子中的每个 M 分配一个 MCache 则会形成不必要的浪费，通过把 cache 从 M 移到 P，每个运行的 M 都有关联的 P，这样只有运行的 M 才有自己的 MCache。

#### 3.Goroutine vs OS thread 有什么区别？
> 其实 goroutine 用到的就是线程池的技术，当 goroutine 需要执行时，会从 thread pool 中选出一个可用的 M 或者新建一个 M。而 thread pool 中如何选取线程，扩建线程，回收线程，Go Scheduler 进行了封装，对程序透明，只管调用就行，从而简化了 thread pool 的使用。


#### 4.sysmon功能是什么？

>释放闲置超过5分钟的span物理内存；
>如果超过2分钟没有垃圾回收，强制执行；
>将长时间未处理的netpoll结果添加到任务队列；
>向长时间运行的G任务发出抢占调度；
>收回因syscall长时间阻塞的P；