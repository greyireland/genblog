---
title: gin概览
tags:
  - gin
categories:
  - 框架
date: 2019-04-05 23:23:09
---

## Gin 框架概览

### 基本用法

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
 r.Run() // listen and serve on 0.0.0.0:8080
}
```

### 实现 middleware

```go
//使用middleware
r := gin.New()
r.Use(gin.Recovery())

//定义middleware
func Recovery() HandlerFunc {
	return func(c *Context) {
		defer func() {
			if err := recover(); err != nil {
			}
		}()
		//fmt.Print("before")
		c.Next()
		//fmt.Print("after")

	}
}

//核心实现，调用下一个handler
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}

type RouterGroup struct {
	Handlers HandlersChain //添加middleware链条
	basePath string //base路径
	engine   *Engine //引用全局engine
	root     bool //是否根
}
r.Use(gin.Recovery()) //添加middleware到Handlers链条
接入和handler一样
type HandlerFunc func(*Context)
```

### 路径分组实现

```go
//使用方式
mux = gin.New()
api := mux.Group("/api/v1")
{
	api.POST("/login", controller.Login)
	api.DELETE("/logout", middleware.SessionCheck(), controller.Logout)
}

//实现
func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup {
	return &RouterGroup{
		Handlers: group.combineHandlers(handlers),//全局的handlers+当前组的handlers
		basePath: group.calculateAbsolutePath(relativePath),//计算相对/路径
		engine:   group.engine,//全局的engine对象
	}
}
```

### 入口

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := engine.pool.Get().(*Context)//并发：多个context
	c.writermem.reset(w) //resp
	c.Request = req //req
	c.reset() //重置gincontext

	engine.handleHTTPRequest(c) //执行handler链条

	engine.pool.Put(c)
}
```
