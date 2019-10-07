---
title: etcd_intro
tags:
  - etcd
categories:
  - etcd
date: 2019-10-07 21:12:21
---

# etcd笔记

> 定义：分布式的kv存储系统

# 常用API

```
get key
put key val
xxx=lease grant 60
put key val —lease=xxx
delete key
watch key
```



#### lease定义一个过期时长，多个key都可以绑定到这个lease上，lease过期后会删除所有关联的key

>  keep-alive让lease不过期

