---
title: net/http包关键点解析
tags:
  - net/http
categories:
  - go
date: 2019-03-03 13:23:19
---

# net/http包关键点解析
## Go创建一个http server
如何创建http server服务端?

核心：

1.保存path和handler的对应关系

2.请求过来时，查找path对应的handler，然后调用handler的ServeHTTP(w, r)方法
```go
package main

import (
    "net/http"
)

func SayHello(w http.ResponseWriter, req *http.Request) {
    w.Write([]byte("Hello"))
}

func main() {
    http.HandleFunc("/hello", SayHello)
    http.ListenAndServe(":8001", nil)

}

```
首先调用Http.HandleFunc

按顺序做了几件事：

1. 调用了DefaultServerMux的HandleFunc
2. 调用了DefaultServerMux的Handle
3. 往DefaultServeMux的map[string]muxEntry中增加对应的handler和路由规则

其次调用http.ListenAndServe(“:8001”, nil)
按顺序做了几件事情：
1. 实例化Server
2. 调用Server的ListenAndServe()
3. 调用net.Listen(“tcp”, addr)监听端口
4. 启动一个for循环，在循环体中Accept请求
对每个请求实例化一个Conn，并且开启一个goroutine为这个请求进行服务go c.serve()
读取每个请求的内容w, err := c.readRequest()
判断header是否为空，如果没有设置handler（这个例子就没有设置handler），handler就设置为DefaultServeMux调用handler的ServeHttp
5. 下面就进入到DefaultServerMux.ServeHttp
根据request选择handler，并且进入到这个handler的ServeHTTP mux.handler(r).ServeHTTP(w, r)选择handler：
    - 判断是否有路由能满足这个request（循环遍历ServerMux的muxEntry）

    - 如果有路由满足，调用这个路由handler的ServeHttp

    - 如果没有路由满足，调用NotFoundHandler的ServeHttp





## Go 发送http请求
步骤：
1. 创建request
2. 创建一个Client将request发送出去，依赖底层RT实现，可以是默认的Transport，也可以是Mock的Transport或者带缓存的Transport


```go
func httpDo() {
    //默认的往返车RT,RoundTripper ，http.Transport默认的网络传输器net.Conn读写
    tr := &http.Transport{
    MaxIdleConns:       10,
    IdleConnTimeout:    30 * time.Second,
    DisableCompression: true,
  }
    client := &http.Client{Transport: tr}
 
    req, err := http.NewRequest("POST", "http://www.baidu.com", strings.NewReader("name=cjb"))
    if err != nil {
        // handle error
    }
    req.Header.Set("Cookie", "name=anny")
 
    resp, err := client.Do(req)
 
    defer resp.Body.Close()
 
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        // handle error
    }
 
    fmt.Println(string(body))
}
```

```go
//带缓存的传输器
func (c *cacheTransport) RoundTrip(r *http.Request) (*http.Response, error) {

    // Check if we have the response cached..
    // If yes, we don't have to hit the server
    // We just return it as is from the cache store.
    if val, err := c.Get(r); err == nil {
        fmt.Println("Fetching the response from the cache")
        return cachedResponse([]byte(val), r)
    }

    // Ok, we don't have the response cached, the store was probably cleared.
    // Make the request to the server.
    resp, err := c.originalTransport.RoundTrip(r)

    if err != nil {
        return nil, err
    }

    // Get the body of the response so we can save it in the cache for the next request.
    buf, err := httputil.DumpResponse(resp, true)

    if err != nil {
        return nil, err
    }

    // Saving it to the cache store
    c.Set(r, string(buf))

    fmt.Println("Fetching the data from the real source")
    return resp, nil
}
//使用：
cachedTransport := newTransport()
// cachedTransport 是自定义实现http.RoundTripper接口的 Transport
client := &http.Client{
    Transport: cachedTransport,
    Timeout:   time.Second * 5,
}

```


### QA问题？
0. net/http相关类介绍？
```go
Server： 服务器类，接收请求后起协程 处理请求：go srv.newConn(rw).serve(ctx)

conn：server的连接实例，负责解析请求，构建Request和Response，并将参数传递给路由Mux处理

DefaultServerMux：默认路由，负责根据path找对应的handler执行，对应的ServeHTTP(w,r)

Handler：业务自己写的逻辑函数，包含一个ServeHTTP(w,r)方法

Request：请求体，包含请求头和请求体等各种参数

Response：返回体，包含请求体，返回的conn实例

Client：客户端client，调用底层传输器发送请求

Transport：Http默认传输器，发送网络请求


```

1. 函数HandlerFunc变成handler?【设计模式之适配器模式】
```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

//适配器模式，函数是一个类，也可以拥有函数
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}

```

2. DefaultServerMux可以替换吗？

可以，如使用httprouter替换默认的DefaultServerMux路由
```go
package main

import (
    "fmt"
    "github.com/julienschmidt/httprouter"
    "net/http"
    "log"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)
    router.GET("/hello/:name", Hello)

    log.Fatal(http.ListenAndServe(":8080", router))
}

```