---
title: golang-dep
tags:
  - tags
categories:
  - categories
date: 2018-12-09 15:48:45
---

# go dep 


### 使用
```
//优先使用本地包初始化【本地获取较快,-v查看安装过程】
dep init -gopath -v

//保证同步
dep ensure

//查看状态【mac】
brew install graphviz
dep status -dot | dot -T png | open -f -a /Applications/Preview.app
```

### 文件说明
![](https://ws1.sinaimg.cn/large/e5320b2aly1fy0k6wfvtpj20pq0ao0tg.jpg)
> `dep ensure` means: Hey dep, please make sure that my project is in sync: that Gopkg.lock satisfies all the imports in my project, and all the rules in Gopkg.toml, and that vendor/ contains exactly what Gopkg.lock says it should."

### 什么时候使用 `dep ensure`
Using dep ensure
There are four times when you'll run dep ensure:

- To add a new dependency
`dep ensure -add github.com/pkg/errors`
- To update an existing dependency
`dep ensure -update`
- To catch up after importing a package for the first time in your project, or removing the last import of a package in your project
- To catch up to a change to a rule in Gopkg.toml




### 参考链接
https://studygolang.com/articles/10589

https://golang.github.io/dep/docs/daily-dep.html