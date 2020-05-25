---
title: etcd_intro
tags:
  - etcd
categories:
  - etcd
date: 2019-10-07 21:12:21
---

## etcd 笔记

> 定义：分布式的 kv 存储系统

## 常用 API

```bash
get key
put key val
xxx=lease grant 60
put key val —lease=xxx
delete key
watch key
```

## 过期时间

lease 定义一个过期时长，多个 key 都可以绑定到这个 lease 上，lease 过期后会删除所有关联的 key

> keep-alive 让 lease 不过期
