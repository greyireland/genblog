---
title: go错误和日志
tags:
  - go
  - error
  - log
categories:
  - go
date: 2019-10-16 10:19:05
---

## 错误处理

在 Go 语言中声明 error 可以有多种方式：

- errors.New 声明包含简单静态字符串的 error
- fmt.Errorf 格式化 error string

- 其他自定义类型使用了 Error() 方法
- 使用 "pkg/errors".Wrap

当要把 error 作为返回值的时候，可以考虑如下的处理方式

- 是不是不需要额外信息，如果是，errors.New 就足够了。（4 星）
- client 需要检测和处理返回的 error 吗？如果是，最好使用实现了 Error() 方法的自定义类型，这样可以包含更多的信息。（3 星）
- error 是不是从下游函数传递过来的？如果是，考虑一下 error wrap。（5 星）
- 其他情况，fmt.Errorf 一般足够了。（5 星）

另外：在直接暴露自定义的 error 类型的时候，最好 export 配套的检测自定义 error 类型的函数。

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

## 日志处理

```go
fmt.Errorf("unsupported signing method: %T", opts.SignMethod)
log.Warn("unknown JWT options", zap.Strings("keys", keys))
log.Errorf("problem loading JWT options: %s", err)
log.Infof("deleting token %s for user %s", tk, username)
log.Errorf("failed to hash password: %s", err)

```
