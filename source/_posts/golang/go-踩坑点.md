---
title: go-踩坑点
date: 2018-09-19 10:53:27
categories: 
- go
tags:
- go 
---
# go遇到的一些问题


###  chan关闭之后，还可以读写吗？
> 不能写，但可以读
~~~
package main

import (
	"fmt"
	"sync"
)

func main() {
	var ret = make(chan []int, 1000)
	var wg = sync.WaitGroup{}
	wg.Add(3)
	go func() {
		defer wg.Done()
		var result = make([]int, 0)
		for i := 0; i < 10; i++ {
			result = append(result, i)
		}
		ret <- result
	}()
	go func() {
		defer wg.Done()
		var result = make([]int, 0)
		for i := 10; i < 20; i++ {
			result = append(result, i)
		}
		ret <- result
	}()
	go func() {
		defer wg.Done()
		var result = make([]int, 0)
		for i := 20; i < 30; i++ {
			result = append(result, i)
		}
		ret <- result
	}()
	wg.Wait()
	//go func() {
	//	for {
	//		ret <- []int{1}
	//		time.Sleep(time.Second)
	//	}
	//}()
	//close(ret)
	for v := range ret { //0 range if v==0 break ,or panic
		for _, v := range v {
			fmt.Println(v)
		}
	}
}

~~~

### 并发获取数据，常见问题
~~~
for _,v:=range uids{
	go func(){
    	fmt.Println(v)//v的值是一样的！
    }()
}
==》
for _,v:=range uids{
	go func(v int){
    	fmt.Println(v)//v的值正常
    }(v)
}
~~~