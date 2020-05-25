---
title: sig信号
tags:
  - signal
categories:
  - linux
date: 2019-10-07 20:33:55
---

## signal 信号

1.信号的处理方式？

> 信号（signal）-- 进程之间通讯的方式，是一种软件中断。一个进程一旦接收到信号就会打断原来的程序执行流程来处理信号。

```py
singnal.signal(signalnum, handler)#

import signal
import time
def myHandler(signum, frame):
    print('I received: ', signum)
if __name__ == "__main__":
    signal.signal(signal.SIGALRM, myHandler)//信号注册
    while True:
        signal.alarm(1)//发送信号
        time.sleep(1)
```

2.进程与线程的区别?

> 进程独享进程空间 fork 全部复制一份，线程共享进程空间，栈上的东西独享
