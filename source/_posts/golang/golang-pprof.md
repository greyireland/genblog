---
title: golang-pprof
tags:
  - tags
categories:
  - categories
date: 2019-01-04 16:30:20
---

## pprof 的使用

什么是 Profiling?
Profiling 这个词比较难翻译，一般译成画像。比如在案件侦破的时候会对嫌疑人做画像，从犯罪现场的种种证据，找到嫌疑人的各种特征，方便对嫌疑人进行排查；还有就是互联网公司会对用户信息做画像，通过了解用户各个属性（年龄、性别、消费能力等），方便为用户推荐内容或者广告。

在计算机性能调试领域里，profiling 就是对应用的画像，这里画像就是应用使用 CPU 和内存的情况。也就是说应用使用了多少 CPU 资源？都是哪些部分在使用？每个函数使用的比例是多少？有哪些函数在等待 CPU 资源？知道了这些，我们就能对应用进行规划，也能快速定位性能瓶颈。

golang 是一个对性能特别看重的语言，因此语言中自带了 profiling 的库，这篇文章就要讲解怎么在 golang 中做 profiling。

在 go 语言中，主要关注的应用运行情况主要包括以下几种：

- CPU profile：报告程序的 CPU 使用情况，按照一定频率去采集应用程序在 CPU 和寄存器上面的数据
- Memory Profile（Heap Profile）：报告程序的内存使用情况
- Block Profiling：报告 goroutines 不在运行状态的情况，可以用来分析和查找死锁等性能瓶颈
- Goroutine Profiling：报告 goroutines 的使用情况，有哪些 goroutine，它们的调用关系是怎样的

两种收集方式

做 Profiling 第一步就是怎么获取应用程序的运行情况数据。go 语言提供了 runtime/pprof 和 net/http/pprof 两个库，这部分我们讲讲它们的用法以及使用场景。

### 工具型应用

如果你的应用是一次性的，运行一段时间就结束。那么最好的办法，就是在应用退出的时候把 profiling 的报告保存到文件中，进行分析。对于这种情况，可以使用 runtime/pprof 库。

pprof 封装了很好的接口供我们使用，比如要想进行 CPU Profiling，可以调用 pprof.StartCPUProfile() 方法，它会对当前应用程序进行 CPU profiling，并写入到提供的参数中（w io.Writer），要停止调用 StopCPUProfile() 即可。

去除错误处理只需要三行内容，一般把部分内容写在 main.go 文件中，应用程序启动之后就开始执行：

```go
f, err := os.Create(\*cpuprofile) ... pprof.StartCPUProfile(f) defer pprof.StopCPUProfile()
```

应用执行结束后，就会生成一个文件，保存了我们的 CPU profiling 数据。

想要获得内存的数据，直接使用 WriteHeapProfile 就行，不用 start 和 stop 这两个步骤了：

```go
f, err := os.Create(\*memprofile) pprof.WriteHeapProfile(f) f.Close()
```

### 服务型应用

如果你的应用是一直运行的，比如 web 应用，那么可以使用 net/http/pprof 库，它能够在提供 HTTP 服务进行分析。

如果使用了默认的 http.DefaultServeMux（通常是代码直接使用 http.ListenAndServe("0.0.0.0:8000", nil)），只需要添加一行：

```go
import \_ "net/http/pprof"
```

如果你使用自定义的 Mux，则需要手动注册一些路由规则：

```go
r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```

不管哪种方式，你的 HTTP 服务都会多出 /debug/pprof endpoint，访问它会得到类似下面的内容：

```go
/debug/pprof/

profiles:
0 block
62 goroutine
444 heap
30 threadcreate

full goroutine stack dump
```

这个路径下还有几个子页面：

```go
/debug/pprof/profile：访问这个链接会自动进行 CPU profiling，持续 30s，并生成一个文件供下载
/debug/pprof/heap： Memory Profiling 的路径，访问这个链接会得到一个内存 Profiling 结果的文件
/debug/pprof/block：block Profiling 的路径
/debug/pprof/goroutines：运行的 goroutines 列表，以及调用关系
```

go tool pprof 命令：获取和分析 Profiling 数据
能通过对应的库获取想要的 Profiling 数据之后（不管是文件还是 http），下一步就是要对这些数据进行保存和分析，我们可以使用 go tool pprof 命令行工具。

在后面我们会生成调用关系图和火焰图，需要安装 graphviz 软件包，在 ubuntu 系统可以使用下面的命令：

```bash
sudo apt-get install -y graphviz
```

NOTE：获取的 Profiling 数据是动态的，要想获得有效的数据，请保证应用处于较大的负载（比如正在生成中运行的服务，或者通过其他工具模拟访问压力）。否则如果应用处于空闲状态，得到的结果可能没有任何意义。

CPU Profiling
go tool pprof 最简单的使用方式为 `go tool pprof [binary] [source]`，binary 是应用的二进制文件，用来解析各种符号；source 表示 profile 数据的来源，可以是本地的文件，也可以是 http 地址。比如：

```go
➜ go tool pprof ./hyperkube http://172.16.3.232:10251/debug/pprof/profile
Fetching profile from http://172.16.3.232:10251/debug/pprof/profile
Please wait... (30s)
Saved profile in /home/cizixs/pprof/pprof.hyperkube.172.16.3.232:10251.samples.cpu.002.pb.gz Entering interactive mode (type "help" for commands) (pprof)
```

这个命令会进行 CPU profiling 分析，等待一段时间（默认是 30s，如果在 url 最后加上 ?seconds=60 参数可以调整采集数据的时间为 60s）之后，我们就进入了一个交互式命令行，可以对解析的结果进行查看和导出。可以通过 help 来查看支持的自命令有哪些。

一个有用的命令是 topN，它列出最耗时间的地方：

```go
(pprof) top10
```

每一行表示一个函数的信息。前两列表示函数在 CPU 上运行的时间以及百分比；第三列是当前所有函数累加使用 CPU 的比例；第四列和第五列代表这个函数以及子函数运行所占用的时间和比例（也被称为累加值 cumulative），应该大于等于前两列的值；最后一列就是函数的名字。如果应用程序有性能问题，上面这些信息应该能告诉我们时间都花费在哪些函数的执行上了。

pprof 不仅能打印出最耗时的地方(top)，还能列出函数代码以及对应的取样数据(list)、汇编代码以及对应的取样数据(disasm)，而且能以各种样式进行输出，比如 svg、gv、callgrind、png、gif 等等。

其中一个非常便利的是 web 命令，在交互模式下输入 web，就能自动生成一个 svg 文件，并跳转到浏览器打开，生成了一个函数调用图：

这个调用图包含了更多的信息，而且可视化的图像能让我们更清楚地理解整个应用程序的全貌。图中每个方框对应一个函数，方框越大代表执行的时间越久（包括它调用的子函数执行时间，但并不是正比的关系）；方框之间的箭头代表着调用关系，箭头上的数字代表被调用函数的执行时间。

因为原图比较大，这里只截取了其中一部分，但是能明显看到 encoding/json.(\*decodeState).object 是这里耗时比较多的地方，而且能看到它调用了哪些函数，分别函数多少。这些更详细的信息对于定位和调优性能是非常有帮助的！

要想更细致分析，就要精确到代码级别了，看看每行代码的耗时，直接定位到出现性能问题的那行代码。pprof 也能做到，list 命令后面跟着一个正则表达式，就能查看匹配函数的代码以及每行代码的耗时：

```go
(pprof) list podFitsOnNode
```

如果想要了解对应的汇编代码，可以使用 disadm <regex> 命令。这两个命令虽然强大，但是在命令行中查看代码并不是很方面，所以你可以使用 weblist 命令，用法和两者一样，但它会在浏览器打开一个页面，能够同时显示源代码和汇编代码。

NOTE：更详细的 pprof 使用方法可以参考 pprof --help 或者 pprof 文档。

Memory Profiling
要想获得内存使用 Profiling 信息，只需要把数据源修改一下就行（对于 http 方式来说就是修改 url 的地址，从 /debug/pprof/profile 改成 /debug/pprof/heap）：

```go
➜ go tool pprof ./hyperkube http://172.16.3.232:10251/debug/pprof/heap
Fetching profile from http://172.16.3.232:10251/debug/pprof/heap
Saved profile in /home/cizixs/pprof/pprof.hyperkube.172.16.3.232:10251.inuse_objects.inuse_space.002.pb.gz Entering interactive mode (type "help" for commands) (pprof)
```

和 CPU Profiling 使用一样，使用 top N 可以打印出使用内存最多的函数列表：

```go
(pprof) top
```

每一列的含义也是类似的，只不过从 CPU 使用时间变成了内存使用大小，就不多解释了。

类似的，web 命令也能生成 svg 图片在浏览器中打开，从中可以看到函数调用关系，以及每个函数的内存使用多少。

需要注意的是，默认情况下，统计的是内存使用大小，如果执行命令的时候加上 --inuse_objects 可以查看每个函数分配的对象数；--alloc-space 查看分配的内存空间大小。

### 常用命令

```go
go tool pprof -http=":60000" '127.0.0.1:27159/debug/pprof/profile'
go tool pprof -http=":60001" '127.0.0.1:27159/debug/pprof/heap'
go tool pprof -http=":60002" '127.0.0.1:36514/debug/pprof/allocs'
```
