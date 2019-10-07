---
title: thrift_intro
tags:
  - thrift
categories:
  - rpc
date: 2019-10-07 20:40:13
---
# thrift笔记

> Thrift实际上是实现了C/S模式，通过代码生成工具将接口定义文件生成服务器端和客户端代码（可以为不同语言），从而实现服务端和客户端跨语言的支持。用户在Thirft描述文件中声明自己的服务，这些服务经过编译后会生成相应语言的代码文件，然后用户实现服务（客户端调用服务，服务器端提服务）便可以了。其中protocol（协议层, 定义数据传输格式，可以为二进制或者XML等）和transport（传输层，定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等）被用作运行时库。

Thrift的网络栈如下所示：

![](https://raw.githubusercontent.com/greyireland/images/master/img/20191007203916.png)

**3.1   Transport**

Transport层提供了一个简单的网络读写抽象层。这使得thrift底层的transport从系统其它部分（如：序列化/反序列化）解耦。以下是一些Transport接口提供的方法：

除了以上几个接口，Thrift使用ServerTransport接口接受或者创建原始transport对象。正如名字暗示的那样，ServerTransport用在server端，为到来的连接创建Transport对象。

**3.2   Protocol**

Protocol抽象层定义了一种将内存中数据结构映射成可传输格式的机制。换句话说，Protocol定义了datatype怎样使用底层的Transport对自己进行编解码。因此，Protocol的实现要给出编码机制并负责对数据进行序列化。

Protocol接口的定义如下：

下面是一些对大部分thrift支持的语言均可用的protocol：

(1)     binary：简单的二进制编码

(2)     Compact：具体见THRIFT-11

(3)     Json

**3.3   Processor**

Processor封装了从输入数据流中读数据和向数据数据流中写数据的操作。读写数据流用Protocol对象表示。Processor的结构体非常简单:

与服务相关的processor实现由编译器产生。Processor主要工作流程如下：从连接中读取数据（使用输入protocol），将处理授权给handler（由用户实现），最后将结果写到连接上（使用输出protocol）。

**3.4   Server**

Server将以上所有特性集成在一起：

（1）  创建一个transport对象

（2）  为transport对象创建输入输出protocol

（3）  基于输入输出protocol创建processor

（4）  等待连接请求并将之交给processor处理



