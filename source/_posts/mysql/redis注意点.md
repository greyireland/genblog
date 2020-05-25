---
title: redis注意点
tags:
  - redis
categories:
  - redis
date: 2019-10-07 20:44:03
---

## redis 关键点

核心点

- 数据类型实现
  - 跳跃表
    - 层级高度
    - 插入、删除过程
  - hash 表
    - rehash 过程
- 持久化
  - RDB
    - BGSAVE
  - AOF
    - BGREWRITEAOF
- 多机
  - 主从模式
    - psync 指令
  - sentinel 主从故障转移
    - 订阅同一个通道
    - 心跳保活
  - redis-cluster 集群
    - 槽 slot 动态分配
    - MOVE、ASK 指令转移
