---
title: protobuf_intro
tags:
  - protobuf
categories:
  - protobuf
date: 2019-10-07 20:55:39
---

## 为什么要使用 protobuf

跟 JSON 相比 protobuf

优点

- 性能更高，更加规范
- 编解码速度快，数据体积小
- 使用统一的规范，不用再担心大小写不同导致解析失败等蛋疼的问题了

缺点

- 改动协议字段，需要重新生成文件。
- 数据没有可读性

## 安装

在 go 中使用 protobuf，有两个可选用的包 goprotobuf（go 官方出品）和 gogoprotobuf。
gogoprotobuf 完全兼容 google protobuf，它生成的代码质量和编解码性能均比 goprotobuf 高一些

## 安装 protoc

首先去[https://github.com/google/pro…](https://github.com/google/protobuf/releases) 上下载 protobuf 的编译器 protoc，同时设置环境变量

## 安装 protobuf 库文件

## goprotobuf

安装插件和依赖库

```go
go get github.com/golang/protobuf/proto
go get github.com/golang/protobuf/protoc-gen-go
```

生成 go 文件

```go
protoc --go_out=. *.proto
```

## gogoprotobuf

安装插件和依赖库

gogoprotobuf 有两个插件可以使用

- protoc-gen-gogo：和 protoc-gen-go 生成的文件差不多，性能稍微快一点点
- protoc-gen-gofast：生成的文件更复杂，性能也更高(快 5-7 倍)

```go
// 依赖库
go get github.com/gogo/protobuf/proto
go get github.com/gogo/protobuf/gogoproto  //这个不装也没关系

//gogo
go get github.com/gogo/protobuf/protoc-gen-gogo

//gofast
go get github.com/gogo/protobuf/protoc-gen-gofast
```

生成 go 文件

```go
//gogo
protoc --gogo_out=. *.proto

//gofast
protoc --gofast_out=. *.proto
```

## 简单使用

test.proto

```proto3
//指定版本，必须要写（proto3、proto2）
syntax = "proto3";
package proto;

message UserInfo{
    string message = 1;   //消息
    int32 length = 2;    //消息大小
    int32 cnt = 3;      //消息计数
}
```

client_protobuf.go

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "os"
    stProto "proto"
    "time"

    //protobuf编解码库,下面两个库是相互兼容的，可以使用其中任意一个
    "github.com/golang/protobuf/proto"
    //"github.com/gogo/protobuf/proto"
)

func main() {
    strIP := "localhost:6600"
    var conn net.Conn
    var err error

    //连接服务器
    for conn, err = net.Dial("tcp", strIP); err != nil; conn, err = net.Dial("tcp", strIP) {
        fmt.Println("connect", strIP, "fail")
        time.Sleep(time.Second)
        fmt.Println("reconnect...")
    }
    fmt.Println("connect", strIP, "success")
    defer conn.Close()

    //发送消息
    cnt := 0
    sender := bufio.NewScanner(os.Stdin)
    for sender.Scan() {
        cnt++
        stSend := &stProto.UserInfo{
            Message: sender.Text(),
            Length:  *proto.Int(len(sender.Text())),
            Cnt:     *proto.Int(cnt),
        }

        //protobuf编码
        pData, err := proto.Marshal(stSend)
        if err != nil {
            panic(err)
        }

        //发送
        conn.Write(pData)
        if sender.Text() == "stop" {
            return
        }
    }
}
```

server_protobuf.go

```go
package main

import (
    "fmt"
    "net"
    "os"
    stProto "proto"

    //protobuf编解码库,下面两个库是相互兼容的，可以使用其中任意一个
    "github.com/golang/protobuf/proto"
    //"github.com/gogo/protobuf/proto"
)

func main() {
    //监听
    listener, err := net.Listen("tcp", "localhost:6600")
    if err != nil {
        panic(err)
    }

    for {
        conn, err := listener.Accept()
        if err != nil {
            panic(err)
        }
        fmt.Println("new connect", conn.RemoteAddr())
        go readMessage(conn)
    }
}

//接收消息
func readMessage(conn net.Conn) {
    defer conn.Close()
    buf := make([]byte, 4096, 4096)
    for {
        //读消息
        cnt, err := conn.Read(buf)
        if err != nil {
            panic(err)
        }

        stReceive := &stProto.UserInfo{}
        pData := buf[:cnt]

        //protobuf解码
        err = proto.Unmarshal(pData, stReceive)
        if err != nil {
            panic(err)
        }

        fmt.Println("receive", conn.RemoteAddr(), stReceive)
        if stReceive.Message == "stop" {
            os.Exit(1)
        }
    }
}
```

## 性能测试

这里只是简单的用 go test 测试了一下

```go
//goprotobuf
"编码"：447ns/op
"解码"：422ns/op

//gogoprotobuf-go
"编码"：433ns/op
"解码"：427ns/op

//gogoprotobuf-fast
"编码"：112ns/op
"解码"：112ns/op
```

## gRPC-go

安装

```go
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
go get -u google.golang.org/grpc
protoc --go_out=plugins=grpc:. *.proto
```

gRPC 仓库

```go
// 引入包
google.golang.org/grpc

// github
https://github.com/grpc/grpc-go

```

gRPC 文档

[gRPC 文档](https://godoc.org/google.golang.org/grpc)

gRPC 需要使用插件：`plugins=grpc`，冒号`:`表示分割，点`.`表示当前目录

```shell
protoc -I/usr/local/include -I. -I$GOPATH/src -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis --go_out=plugins=grpc:. *.proto
```

protoc 工作原理

[protoc 原理](<https://hitzhangjie.github.io/2017/05/23/Protoc%E5%8F%8A%E6%8F%92%E4%BB%B6%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90(%E7%B2%BE%E5%8D%8E%E7%89%88).html>)

gRPC gateway
[gRPC 仓库](https://github.com/grpc-ecosystem/grpc-gateway)
优点

- http rest 调用方式
- swagger 文档生成
