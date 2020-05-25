---
title: go-test简介
tags:
  - test
  - bench test
categories:
  - go
date: 2019-03-13 11:16:46
---

## golang 测试基本用法

### 入门

```go
package main

import (
	"fmt"
	"strings"
	"bytes"
)

func StringPlus() string {
	var s string
	s += "昵称" + ":" + "greyireland" + "\n"
	s += "博客" + ":" + "greyireland.com" + "\n"
	s += "微信公众号" + ":" + "没有"
	return s
}
func StringFmt() string {
	return fmt.Sprint("昵称", ":", "greyireland", "\n", "博客", ":", "greyireland.com", "\n", "微信公众号", ":", "没有")
}
func StringJoin() string {
	s := []string{"昵称", ":", "greyireland", "\n", "博客", ":", "greyireland.com", "\n", "微信公众号", ":", "没有"}
	return strings.Join(s, "")
}

func StringBuffer() string {
	var b bytes.Buffer
	b.WriteString("昵称")
	b.WriteString(":")
	b.WriteString("greyireland")
	b.WriteString("\n")
	b.WriteString("博客")
	b.WriteString(":")
	b.WriteString("greyireland.com")
	b.WriteString("\n")
	b.WriteString("微信公众号")
	b.WriteString(":")
	b.WriteString("没有")
	return b.String()
}
```

测试源码【goland 可以使用 cmd+shift+T 快键键生成测试用例】

```go
package main

import (
	"testing"
)

func BenchmarkStringPlus(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringPlus()
	}
}
func BenchmarkStringJoin(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringJoin()
	}
}
func BenchmarkStringBuffer(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringBuffer()
	}
}
func BenchmarkStringFmt(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringFmt()
	}
}

func TestStringJoin(t *testing.T) {
	tests := []struct {
		name string
		want string
	}{
		{"", "昵称:greyireland\n博客:greyireland.com\n微信公众号:没有"},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := StringJoin(); got != tt.want {
				t.Errorf("StringJoin() = %v, want %v", got, tt.want)
			}
		})
	}
}

func TestStringBuffer(t *testing.T) {
	tests := []struct {
		name string
		want string
	}{
		{"", "昵称:greyireland\n博客:greyireland.com\n微信公众号:没有"},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := StringBuffer(); got != tt.want {
				t.Errorf("StringBuffer() = %v, want %v", got, tt.want)
			}
		})
	}
}
//基准测试参数解释
// func setUp() *bolt.DB {
// 	db, _ := bolt.Open("my.db", 0600, nil)
// 	return db
// }
func BenchmarkWriteKV(b *testing.B) {
	// setUp()
	b.ResetTimer()       //重置前面的耗时
	b.SetParallelism(10) //设置并发因子
	b.ReportAllocs()     //报告内存
	b.SetBytes(24004)    //设置一次执行的内存大小，评估吞吐量
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() { //一直循环执行
			fmt.Sprintf("%v", 1)
		}
	})
}
//BenchmarkWriteKV-4      30000000                70.2 ns/op      342031.82 MB/s         0 B/op          0 allocs/op

```

### 常见执行命令

```go
// 基本测试
go test -v  //执行所有Test方法
go test -v -run=^TestStringBuilder$ //只执行某个函数测试
示例：
➜  go-test go test -v -run=^TestStringFmt$
=== RUN   TestStringFmt
--- PASS: TestStringFmt (0.00s)
PASS
ok      AAA/go-test     0.008s


//基准测试
go test -v bench=. -benchmem //执行所有基准测试用例
示例：
➜  go-test go test -v -bench=String -benchmem -cover
goos: darwin
goarch: amd64
pkg: AAA/go-test
BenchmarkStringPlus-4           10000000               125 ns/op             144 B/op          2 allocs/op
BenchmarkStringJoin-4           10000000               194 ns/op             160 B/op          2 allocs/op
BenchmarkStringBuffer-4          5000000               349 ns/op             336 B/op          3 allocs/op
BenchmarkStringFmt-4             5000000               390 ns/op              80 B/op          1 allocs/op
PASS
coverage: 100.0% of statements
ok      AAA/go-test     8.019s

说明：
-benchmem显示内存使用  -cover显示代码覆盖率
390 ns/op //一次操作耗时
80 B/op //一次内存分配占用大小
1 allocs/op //一次操作几次内存分配
```

最常用命令

```go
go test -v -run=^TestStringBuilder$
go test -v bench=^TestStringBuilder$ -benchmem
```

### 更深入了解

[assert](https://godoc.org/github.com/stretchr/testify/assert)
