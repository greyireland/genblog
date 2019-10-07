---
title: mongodb_intro
tags:
  - mongo
categories:
  - db
date: 2019-10-07 20:18:50
---
# mongodb

### 概念

```
# 对比
mysql  MongoDB
数据库	数据库Db
表	   集合Collection
行	   文档Document
列	   字段Key
表     集合[{"k1":"v1","k2":"v2"},{}]

# 常见API
db.collection.insert(doc)
```

### 优势
- 格式不固定JSON
- MapReduce复杂的聚合查询
- 支持全文索引
- 支持分片分布式存储

### 缺点
- 索引需要手动创建
- 索引在内存创建（内存不够，则不会创建索引，或者删除部分索引）


