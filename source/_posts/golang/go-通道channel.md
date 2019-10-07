---
title: go-通道channel
date: 2018-09-19 10:52:42
categories: 
- go
tags:
- go
---
# 通道

两种配合方式：
- range
- select

### range 方式
~~~
//关闭之后会跳出循环
for v:=range c{
	//todo
}
~~~

### select 方式
~~~
for {
	select {
    	case val,ok:=<-c:
        	if !ok{break}
    }
}
~~~

### Close(ch)含义
给通道写了一个结束标识值，接收到这个值表示通道已经关闭，有这个值就不能再写入了，还可以继续读取，读取到这个值的时候就表示已经读取完了，通道为空了。

### 注意点
- 从一个nil channel中接收数据会一直被block。

- 从一个被close的channel中接收数据不会被阻塞，而是立即返回，接收完已发送的数据后会返回元素类型的零值(zero value)。

~~~
//如果OK 是false，表明接收的x是产生的零值，这个channel被关闭了或者为空。
//从这个关闭的channel中不但可以读取出已发送的数据，还可以不断的读取零值(使用range或者判断是否关闭可以跳出)
x, ok := <-ch
~~~