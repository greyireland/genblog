---
title: golang闭包
tags:
  - 闭包
categories:
  - golang
date: 2019-03-16 18:54:36
---

## 闭包

### 关键点

- 闭包也是类，运行闭包等同于创建对象【类是数据附带行为，闭包是行为附带数据】
- 区分全局变量和局部变量

```go
package main

import "fmt"

func add() func() int {
	sum := 1//全局变量
	return func() int {
		var a = 1//局部变量
		sum += a
		return sum
	}
}

func main() {
	pos, neg := add(), add()
	for i := 0; i < 10; i++ {
		fmt.Println(pos(), neg())
	}

}

```
