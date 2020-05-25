---
title: golang入门笔记
tags:
  - go
categories:
  - go
date: 2017-10-07 21:10:08
---

## GO 笔记

### 基本库和概念

```go
//*T 可以传给值接收，也可以传给指针接收，所以传入尽量*T,接收也尽量用*T呗
//使用接口：1.定义接口 2.定义类 3.类实现接口
//a:=[2]byte{'a','1',2}//byte ascii 1 2等这些是显示不出来的，他和'1','2'不一样，后面ascii码：31 32

//传入的所有参数都是interface{}，传出的所有参数也是interface{} 所以出来的时候必须要转为所需要的类型
```

### 模块 类概念

go 的包和文件夹名是一样的（跟文件名关系不大）

> java:com.alibaba.dubbo.xxx
>
> c++:namespace std
>
> go: package 包

### 类

继承、封装、多态

```go
type Person struct{
  Book //==组合继承
  name string
  age int
}

//构造函数-一般自己创建构造函数
func NewPerson(){
  return &Person{Book{},name:"jack",age:19}
}

//类函数外置
func (p *Person)say(){//小写函数 包内访问==封装
  p.name="jack ma"//指针(引用)接受者和普通
}


//==多态
type Flyable interface {
	Fly()
}

type Duck struct{
	Flyable//隐式继承接口
}
type Cat struct {

}

func (t *Duck)Fly()  {
	fmt.Println("duck flying")
}
func (t *Cat)Fly()  {//隐式继承接口
	fmt.Println("cat flying")
}
```

### 嵌入类型

包含嵌入类型所有的财产（你是我的，你所有东西都是我的）

```go
类型
type 类型名字 底层类型
1.代表类型别名，附带新方法
2.代表新类型

方法
包方法-静态方法
类型方法-实例方法
```

```go
func main()  {
	a:=&A{}
	fmt.Println(a,)
	fmt.Println(a.Age)
	fmt.Println(a.Dd.Birth)//初始化为空对象
	//fmt.Println(a.Ee.Gender)//初始化为空指针，拿不到实例的熟悉，所以报错
	a.Ee.Efunc() //空指针可以调用方法
}
type A struct{//
	B //嵌入B类型所有财产
	*C //这里和嵌入C类型区别是，指针类型不能调用其属性
	Dd D //嵌入D类型的实例，有实例了什么都可以干(嵌入实例和嵌入指针比较多一些)
	Ee *E //嵌入E类型的指针，有了指针天下无敌(嵌入实例和嵌入指针比较多一些)
	Name string //本身自带的财产，自己的属性(属性一般是大写，大写导出和c一样私有字段set()get())
  	CheckArg func(*context.Context) error //属性的类型的是一个func，可以直接调用a.CheckArg()
}
type B struct{
	Age int
}
type C struct{
	Email string
}

func (*C)Cfunc()  {
	fmt.Println("c func")
}
type D struct{
	Birth string
}
type E struct{
	Gender bool
}

func (*E)Efunc()  {
	fmt.Println("e func")
}
```

### 可变参数

...interface{}，传入的当做[]interface{}切片

```go
func Println(vals ...interface{}){
  fmt.Println(vals)
}
类似：
func Println(vals []interface{}){}
```

### http.Request 请求

服务端需要 Request,和 Response 往里面写数据

包括：

1.第一行

2.请求头

3.请求体（一般 post 才有）

```go
resp:=DefaultTransport.send(req)//创建一个连接，然后通过这条连接发送req数据，服务端通过这条连接读取req然后构建resp，（也通过resp可以拿到连接的标识，去某个地方获取到这条连接），然后通过连接把resp发送回去
```

### http.Response 响应

客户端需要 Response 从里面读数据，读完了就关闭

包括：

1.第一行数据

2.返回头

3.返回 body

4.请求实例 http.Request

可拿到底层连接 net.Conn(可读可写)

### 读写锁

```go
func (rw *RWMutex) Lock()　　写锁，如果在添加写锁之前已经有其他的读锁和写锁，则lock就会阻塞直到该锁可用，为确保该锁最终可用，已阻塞的 Lock 调用会从获得的锁中排除新的读取器，即写锁权限高于读锁，有写锁时优先进行写锁定
func (rw *RWMutex) Unlock()　写锁解锁，如果没有进行写锁定，则就会引起一个运行时错误

func (rw *RWMutex) RLock() 读锁，当有写锁时，无法加载读锁，当只有读锁或者没有锁时，可以加载读锁，读锁可以加载多个，所以适用于＂读多写少＂的场景

func (rw *RWMutex)RUnlock()　读锁解锁，RUnlock 撤销单次RLock 调用，它对于其它同时存在的读取器则没有效果。若 rw 并没有为读取而锁定，调用 RUnlock 就会引发一个运行时错误(注：这种说法在go1.3版本中是不对的，例如下面这个例子)。
```

### context

三种 ctx

```go
timerCtx:超时的ctx，超过某个时间就close(chan)
cancelCtx:可取消的ctx，使用close(chan)
valueCtx:保存键值的ctx
```

### 并发、channel、select、sync

超时控制

```go
taskdone:=make(chan struct{},1)
//1.任务异步做，做完发一个taskdone消息
go func() {
  //dosomething
  time.Sleep(time.Second)
  taskdone<- struct {}{}
}()
//2.等待taskdone 或者等待超时
select {
case <-taskdone:
	fmt.Println("task done")
case <-time.After(time.Millisecond*5000):
	fmt.Println("task timeout")
}
```

### scanf 扫描

一行一行读取数据嘛，或者一个个字符的读取

读完之后就丢弃掉数据

```go
buf:=bytes.NewBuffer(sli)//buffer读完之后就丢弃掉数据
data0,_,_:=buf.ReadRune()
fmt.Println(string(data0),buf.String())
```

### go time

```go
转为ts:t.unix()
转为str:t.Format(layout,val)
转为Time:time.parse(str,layout)
```

### json,xml,gob

```go
var network bytes.Buffer
enc:=gob.NewEncoder(&network)//编码到什么地方去
enc.Encode(map[string]string{"k1": "v1"})//编码什么东西
fmt.Println(network.Bytes())

dec:=gob.NewDecoder(&network)//从什么地方解码
var m map[string]string
dec.Decode(&m)//解码到哪里
fmt.Println(m)
```

### 字符串拼接

```go
a+="hello"
var b bytes.Buffer
for condition {
    b.WriteString(str) // 将字符串str写入缓存buffer
}
    return b.String()
```

### 指针

**永远不要使用一个指针指向一个接口类型，因为它已经是一个指针。**

```go
write(w io.Writer)//没有w *io.Writer这种
```

### 包管理

```go
自己clone
mkdir -p github.com/user
git clone repo
cd github.com/user/repo
go build xxx//缺啥去下载啥
```

```go
golang.org/x/net ，其实镜像托管在在 github.com/golang/net
你可以用 go get github.com/golang/net 之后，到 GOPATH/src 中
mv github.com/golang/net golang.org/x/net

golang.org/x/text
go get github.com/golang/text
mv github.com/golang/text golang.org/x/text
```

```go
mkdir -p $GOPATH/src/golang.org/x/
cd !$
git clone https://github.com/golang/net.git
git clone https://github.com/golang/sys.git
git clone https://github.com/golang/tools.git
```

### http 参数解析

```go
r.ParseForm()
uid :=r.Form.Get("uid")//get post/form-data
fmt.Println(uid)
body,_:=ioutil.ReadAll(r.Body)//json post
fmt.Println(string(body))
```

### error 错误处理

```go
//1.最普通
if err!=nil{
  return result,err
}
//2.recover() panic()
defer func(){
  if err:=recover();err!=nil{
    //dosomething()
  }
}
panic()

//有些错误可以处理，有些选择处理

```

### 默认值和 nil

```go
bool      -> false
numbers -> 0
string    -> ""

pointers -> nil
slices -> nil
maps -> nil
channels -> nil
functions -> nil
interfaces -> nil

//可以使用len(slice)==0
```

### TCP 和 UDP

```go
//tcp
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)
func (l *TCPListener) Accept() (c Conn, err os.Error)
func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)

//udp
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)

//通用Conn, PacketConn and Listener
net.Dial("tcp",addr)
```

```go
func main() {
  encoder := json.NewEncoder(conn)
  decoder := json.NewDecoder(conn)
  for n := 0; n < 10; n++ {
    encoder.Encode(person)//类似于write
    var newPerson Person
    decoder.Decode(&newPerson)//类似于read
    fmt.Println(newPerson.String())
  }
}
```

```go
/**
* Base64
*/
package main
import (
	"bytes"
	"encoding/base64"
	"fmt"
)
func main() {
	eightBitData := []byte{1, 2, 3, 4, 5, 6, 7, 8}
	bb := &bytes.Buffer{}
	encoder := base64.NewEncoder(base64.StdEncoding, bb)//转为base64字符串
	encoder.Write(eightBitData)
	encoder.Close()
	fmt.Println(bb)
	dbuf := make([]byte, 12)
	decoder := base64.NewDecoder(base64.StdEncoding, bb)
	decoder.Read(dbuf)
	for _, ch := range dbuf {
		fmt.Print(ch)
	}
}
```

### 细节点

```go
select{}//阻塞

```

```go
SERVE_HTTP=":3030" go run 43.go
address = os.Getenv("SERVE_HTTP")//直接从上面拿
```

默认类型

```go
nil 是 interface、function、pointer、map、slice 和 channel 类型变量的默认初始值
```

go 修改值全部用指针

切片，map 先 make，再取地址比较好

问题代码

```go
//下面是一个坑
//for _, v := range *userInfos {
//	userInfoTempMap[v.User.ID] = &v
//}

```

```go
func fillUserInfo(queryUids *[]int64, userInfoTempMap map[int64]*model.RUserInfo, selfU int64) {
	if queryUids == nil || len(*queryUids) == 0 {
		return
	}
	//获取用户信息
	userInfos, err := manager.MGetUserInfo(queryUids, selfU)
	if err != nil || userInfos == nil {
		log.Errorf("get user info error:|%#v|", err)
		return
	}
	//创建临时map给后面直接使用
	for i:=0;i<len(*userInfos);i++{
		userInfoTempMap[(*userInfos)[i].User.ID] = &(*userInfos)[i]
	}
	for _, v := range *userInfos {
		userInfoTempMap[v.User.ID] = &v
	}
	fmt.Println(userInfoTempMap)
}
```

## 需要掌握的点

- http 请求
- http 服务
- json 解析
- 日志
- mysql 库
- redis 库
- list 和 map 使用
- goroutine 使用
- 网络框架使用
