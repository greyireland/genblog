---
title: go-bytes.Buffer
tags:
  - buffer
categories:
  - go
date: 2018-09-19 15:46:07
---
# bytes.Buffer

## java StringBuilder

```java
StringBuilder sb = new StringBuilder();
sb.append("hello");
sb.append(",world");
System.out.println(sb.toString());
```

## go bytes.Buffer

```go
var bb bytes.Buffer
bb.WriteString("[")
bb.WriteString("]")
fmt.Println(bb.String())
```