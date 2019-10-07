---
title: epoll
tags:
  - epoll
categories:
  - cs
date: 2019-06-28 16:56:03
---

# epoll

### epoll如何实现百万连接

```
int epoll_create(int size);//size fd数量,创建了红黑树和就绪链表
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);//添加回调函数到红黑树，事件来临时将回调函数写到就绪链表
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);//取出就绪链表数据
```



#### VS select & poll

轮询实现效率低，poll去除fd限制，实际问题未解决



### 网络IO模型

![](https://raw.githubusercontent.com/greyireland/images/master/img/20190628165226.png)



参考：

<https://zhuanlan.zhihu.com/p/21378825>

<https://www.zhihu.com/question/20831000>

<https://www.jianshu.com/p/55eb83d60ab1>