---
title: go-文件io
date: 2018-09-19 10:47:35
categories:
  - go
tags:
  - go
---

## golang 文件读写

### 使用 io/ioutil 进行读写文件

io/ioutil 其中提到了两个方法

func ReadFile

`func ReadFile(filename string) ([]byte, error)`

ReadFile reads the file named by filename and returns the contents. A successful call returns err == nil, not err == EOF. Because ReadFile reads the whole file, it does not treat an EOF from Read as an error to be reported.

func WriteFile

`func WriteFile(filename string, data []byte, perm os.FileMode) error`
WriteFile writes data to a file named by filename. If the file does not exist, WriteFile creates it with permissions perm; otherwise WriteFile truncates it before writing.

读文件：

```go
package main

import (
    "fmt"
    "io/ioutil"
)

func main() {
    b, err := ioutil.ReadFile("test.log")
    if err != nil {
        fmt.Print(err)
    }
    fmt.Println(b)
    str := string(b)
    fmt.Println(str)
}
```

写文件：

```go
package main

import (
    "io/ioutil"
)

func check(e error) {
    if e != nil {
        panic(e)
    }
}

func main() {

    d1 := []byte("hello\ngo\n")
    err := ioutil.WriteFile("test.txt", d1, 0644)
    check(err)
}
```

### 使用 os 进行读写文件

os 包打开文件的方法

func Open

`func Open(name string) (*File, error)`

Open opens the named file for reading. If successful, methods on the returned file can be used for reading; the associated file descriptor has mode O_RDONLY. If there is an error, it will be of type \*PathError.

读文件：

```go
file, err := os.Open(path)
if err != nil {
    panic(err)
}
defer file.Close()
```

func OpenFile
需要提供文件路径、打开模式、文件权限

`func OpenFile(name string, flag int, perm FileMode) (*File, error)`

OpenFile is the generalized open call; most users will use Open or Create instead. It opens the named file with specified flag (O_RDONLY etc.) and perm, (0666 etc.) if applicable. If successful, methods on the returned File can be used for I/O. If there is an error, it will be of type \*PathError.

读文件：

```go
package main

import (
    "log"
    "os"
)

func main() {
    f, err := os.OpenFile("notes.txt", os.O_RDWR|os.O_CREATE, 0755)
    if err != nil {
        log.Fatal(err)
    }
    if err := f.Close(); err != nil {
        log.Fatal(err)
    }
}
```

读方法

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "io/ioutil"
    "os"
)

func check(e error) {
    if e != nil {
        panic(e)
    }
}

func main() {

    f, err := os.Open("/tmp/dat")
    check(err)

    b1 := make([]byte, 5)
    n1, err := f.Read(b1)

    check(err)
    fmt.Printf("%d bytes: %s\n", n1, string(b1))

    o2, err := f.Seek(6, 0)
    //n1, err := f.ReadAt(b1,offset)
    check(err)
    b2 := make([]byte, 2)
    n2, err := f.Read(b2)
    check(err)
    fmt.Printf("%d bytes @ %d: %s\n", n2, o2, string(b2))

    o3, err := f.Seek(6, 0)
    check(err)
    b3 := make([]byte, 2)
    n3, err := io.ReadAtLeast(f, b3, 2)
    check(err)
    fmt.Printf("%d bytes @ %d: %s\n", n3, o3, string(b3))

    _, err = f.Seek(0, 0)
    check(err)

    r4 := bufio.NewReader(f)
    b4, err := r4.Peek(5)
    check(err)
    fmt.Printf("5 bytes: %s\n", string(b4))

    f.Close()

}
```

> 寻址取偏移量：`func (f *File) Seek(offset int64, whence int) (ret int64, err error)`
> // Seek sets the offset for the next Read or Write on file to offset,interpreted
> // according to whence: 0 means relative to the origin of the file, 1 means
> // relative to the current offset, and 2 means relative to the end.
> // It returns the new offset and an error, if any.
> // The behavior of Seek on a file opened with O_APPEND is not specified.

写方法

```go
package main

import (
    "bufio"
    "fmt"
    "io/ioutil"
    "os"
)

func check(e error) {
    if e != nil {
        panic(e)
    }
}

func main() {

    f, err := os.Create("/tmp/dat2")
    check(err)

    defer f.Close()

    d2 := []byte{115, 111, 109, 101, 10}
    n2, err := f.Write(d2)
    check(err)
    fmt.Printf("wrote %d bytes\n", n2)

    n3, err := f.WriteString("writes\n")
    fmt.Printf("wrote %d bytes\n", n3)
    f.Sync()
    w := bufio.NewWriter(f)
    n4, err := w.WriteString("buffered\n")
    fmt.Printf("wrote %d bytes\n", n4)

    w.Flush()

}
```

几种读取文件方法速度比较

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "io/ioutil"
    "os"
    "time"
)

func read0(path string) string {
    f, err := ioutil.ReadFile(path)//底层调用file.read(fileSizeBuf)
    if err != nil {
        fmt.Printf("%s\n", err)
        panic(err)
    }
    return string(f)
}

func read1(path string) string {
    fi, err := os.Open(path)
    if err != nil {
        panic(err)
    }
    defer fi.Close()

    chunks := make([]byte, 1024, 1024)
    buf := make([]byte, 1024)
    for {
        n, err := fi.Read(buf)//一块一块读
        if err != nil && err != io.EOF {
            panic(err)
        }
        if 0 == n {
            break
        }
        chunks = append(chunks, buf[:n]...)
    }
    return string(chunks)
}

func read2(path string) string {
    fi, err := os.Open(path)
    if err != nil {
        panic(err)
    }
    defer fi.Close()
    r := bufio.NewReader(fi)

    chunks := make([]byte, 1024, 1024)

    buf := make([]byte, 1024)
    for {
        n, err := r.Read(buf)//缓存读
        if err != nil && err != io.EOF {
            panic(err)
        }
        if 0 == n {
            break
        }
        chunks = append(chunks, buf[:n]...)
    }
    return string(chunks)
}

func read3(path string) string {
    fi, err := os.Open(path)
    if err != nil {
        panic(err)
    }
    defer fi.Close()
    fd, err := ioutil.ReadAll(fi)//底层实现使用bytes.Buffer缓存读(和io.Reader/io.Writer一样)
    return string(fd)
}

func main() {

    file := "test.log"

    start := time.Now()

    read0(file)
    t0 := time.Now()
    fmt.Printf("Cost time %v\n", t0.Sub(start))

    read1(file)
    t1 := time.Now()
    fmt.Printf("Cost time %v\n", t1.Sub(t0))

    read2(file)
    t2 := time.Now()
    fmt.Printf("Cost time %v\n", t2.Sub(t1))

    read3(file)
    t3 := time.Now()
    fmt.Printf("Cost time %v\n", t3.Sub(t2))

}
```

运行结果：

```go
Cost time 4.0105ms
Cost time 11.5043ms
Cost time 7.0042ms
Cost time 2.4983ms
```

## 底层实现

### 底层 IO

```go
//os提供的功能
file=os.open(path)
file.read(buf)
file.write(buf)
file.readAt(buf,offset)
file.writeAt(buf,offset)
```

### 缓冲 IO

```go
bufio.Reader/Writer
file=open(path)
bufFile=bufio.NewReader(file)
bufFile.read(buf)

file=open(path)
bufFile = bufio.NewWriter(file)
bufFile.Write([]byte("haha"))
w.Flush()//将bufFile里面的数据刷到file里面去，操作系统可能还有一层buf！
```

> 标准 IO 操作数据流向路径：数据—>进程缓冲（用户态）—>内核缓存区（内核态）—>磁盘
>
> 为什么包一层 buf，buf 读的时候读一大块，给你读取的时候，你只需要从 buf 里面去读一点数据，下次再读一点数据，不用每次读取都去调用系统库，buf 写的时候，当写满一大块的时候，才真正调用系统写，因为不用每次写都去调用系统写，这样会提高性能，但数据可能丢失或是不一致的情况

## 任务

内存大小为 4G 的电脑给 10G 的文件排序？
