---
title: mysql常见问题
tags:
  - mysql
categories:
  - mysql
date: 2019-10-07 21:03:03
---

# mysql原理

### 事务ACID如何实现？

```
redo log重做日志其实保障的是事务的持久性和一致性，而undo log撤销日志则保障了事务的原子性，锁保证隔离性。
(一个事务)写操作对(另一个事务)写操作的影响：锁机制保证隔离性
(一个事务)写操作对(另一个事务)读操作的影响：MVCC保证隔离性
原子性：使用 undo log ，从而达到回滚
持久性：使用 redo log，从而达到故障后恢复
隔离性：使用锁以及MVCC,运用的优化思想有读写分离，读读并行，读写并行
一致性：通过回滚，以及恢复，和在并发环境下的隔离做到一致性。
```

### 索引如何存储？

```
B+树，每一个节点为一页，IO次数约等于树高度
```

### mysql笔记

- uid使用bigint 长度20和primary key 和auto_increment
- create_time 使用timestamp默认值CURRENT_TIMESTAMP
- update_time使用timestamp默认值CURRENT_TIMESTAMP加on_update
- 一般用varchar长度64，128，512最大65535
- 超大用text



### 查询数据库磁盘占用

```sql
select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;
```

### 查询某个数据库每个表磁盘占用

```sql
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables where TABLE_SCHEMA = 'red_packet'
group by TABLE_NAME
order by data_length desc;
```

