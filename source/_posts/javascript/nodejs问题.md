---
title: nodejs问题
tags:
  - tags
categories:
  - categories
date: 2019-10-07 20:59:21
---
Nodejs踩坑

1.执行函数后面记得加分号结束;

```js
function hello(uid,callback)

hello(uid,function(){});//一定要加;

```

functionfunction

2.判断条件

```js
a = []
b = ""
c = {}
d = 0
f = false
g="0"
h=undefined
i=null
//{}对象
if (a) {
    console.info("a")
}
if (a.length) {
    console.info("a.length")
}
if (b) {
    console.info("b")
}
if (b.length) {
    console.info("b.length")
}
//[]也是对象
if (c) {
    console.info("c")
}

if (d) {
    console.info("d")
}
if(d.length) {
    console.info("d.length")
}

if (f) {
    console.info("f")
}
if (f.length) {
    console.info("f.length")
}
//字符串
if (g) {
    console.info('g')
}
//转为0了
if(parseInt(g)){
    console.info('parseInt g')
}
if (h) {
    console.info('h')
}
if (i) {
    console.info('i')
}


//a
//c
//g

```

