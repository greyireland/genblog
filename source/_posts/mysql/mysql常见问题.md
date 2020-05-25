---
title: mysql常见问题
tags:
  - mysql
categories:
  - mysql
date: 2019-10-07 21:03:03
---

## mysql 原理

### 事务 ACID 如何实现

redo log 重做日志其实保障的是事务的持久性和一致性，而 undo log 撤销日志则保障了事务的原子性，锁保证隔离性。
一个事务写操作对另一个事务写操作的影响：锁机制保证隔离性
一个事务写操作对另一个事务读操作的影响：MVCC 保证隔离性

- 原子性：使用 undo log ，从而达到回滚
- 持久性：使用 redo log，从而达到故障后恢复
- 隔离性：使用锁以及 MVCC,运用的优化思想有读写分离，读读并行，读写并行
- 一致性：通过回滚，以及恢复，和在并发环境下的隔离做到一致性。

### 索引如何存储

B+树，每一个节点为一页，IO 次数约等于树高度

### mysql 笔记

- uid 使用 bigint 长度 20 和 primary key 和 auto_increment
- create_time 使用 timestamp 默认值 CURRENT_TIMESTAMP
- update_time 使用 timestamp 默认值 CURRENT_TIMESTAMP 加 on_update
- 一般用 varchar 长度 64，128，512 最大 65535
- 超大用 text

### 总结点

mysql

- 索引
  - B+树
  - 查找页+二分查找
- 锁
  - 分类：SX 读写锁
  - 算法
    - record lock 记录锁
    - gap lock 间隙锁
    - next-key lock 记录间隙锁
- 事务
  - A 原子性
    - redo、undo
  - C 一致性
    - redo、undo、记录间隙锁+MVCC
  - I 隔离性
    - 记录间隙锁+MVCC
  - D 持久性
    - redo、undo
- 隔离级别
  - 未提交读
  - 提交读
  - 重复读
    - 解决重复数据读取不一致
  - 串行化
    - 解决幻读（两次读 出现数据增多 insert 进来的数据）
- 性能调优
  - 表结构优化
  - 索引优化
  - 查询语句优化
  - 配置文件优化
  - 机器优化

### 其他

查询数据库磁盘占用

```sql
select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;
```

查询某个数据库每个表磁盘占用

```sql
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables where TABLE_SCHEMA = 'red_packet'
group by TABLE_NAME
order by data_length desc;
```
