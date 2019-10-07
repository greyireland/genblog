---
title: redis注意点
tags:
  - redis 
categories:
  - redis
date: 2019-10-07 20:44:03
---
# redis注意点

```
//这个max在前面
ZREVRANGEBYSCORE key max min WITHSCORES 
//这个max在后面
ZRANGEBYSCORE key min max WITHSCORES
```

