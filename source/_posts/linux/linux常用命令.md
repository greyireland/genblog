---
title: linux 服务器常用命令整理
date: 2018-07-21 21:24:15
categories:
  - linux
tags:
  - linux
  - cmd
---

## linux 服务器常用命令整理

### 目录

- **网络分析 - tcpdump \ telnet \ (netstat \ ss \ lsof) \ nload**
- **网络传输 - scp \ rsync \ (rz \ sz) \ nc**
- **抓包工具 - charles**
- **内存检查 - free \ meminfo**
- **系统监控 - vmstat \ iostat \ top \ ps \ sar \ dstat**
- **系统调用追踪 - strace \ gcore**
- **文件相关 - find \ awk \ sed \ grep \ tail \ df \ du \ locate**
- **开发效率 - tmux**

### 常见命令

#### tcpdump

1. tcp:用来过滤数据报的类型
2. -i eth1 : 只抓经过接口 eth1 的包
3. -t : 不显示时间戳
4. -s 0 : 抓取数据包时默认抓取长度为 68 字节, 加上-S 0 后可以抓到完整的数据包
5. -c 100 : 只抓取 100 个数据包
6. dst port !22: 不抓取目标端口是 22 数据包
7. src net 10.99.184.0/24 : 数据包的源网络地址为 10.99.184.0/24
8. -A：显示数据包内容

示例：

`tcpdump -i any -v port 8888`

`tcpdump -i any -A port 8888`

sudo tcpdump port 17280 -i lo0 -X

#### netstat

查看所有连接

`netstat -autnp`

查看监听的 tcp 服务

`netstat -altnp`

看 tcp 端口

`netstat -ltnp`

#### ss

- `ss -pl` 查看每个进程及其监听的端口
- `ss -t -a` 查看所有的 tcp 连接
- `ss -u -a` 查看所有的 udp 连接

#### lsof

- `lsof -i :8888` 查看端口 8888 进程信息
- `lsof -p 7915` 查看进程 7915 打开的 fd 信息

#### scp

- `scp -r src remote:/tmp` 本地拷贝到远端
- `scp -r remote:/tmp/src .` 远端拷贝到本地
- `scp -3 remote:/tmp/a.tar remote2:/tmp/` 以本地为跳板机，将 remote 机器上文件拷贝到 remote2

#### rsync

- `rsync -av /home/mail/ 192.168.11.12:/home/mail/`
- `rsync -av 192.168.11.11:/home/mail/ /home/mail/`

#### nc

- `nc -l 8888` 本地启动 8888 端口
- `nc -l 8888 > a.tgz` 接收文件
- `nc ali-a-bpc-userdevelop01.bj:8888 < a.tgz` 发送文件到远端

#### vmstat

- `vmstat 1 10`对内存监控，重点关注 swpd、free、si、so。一般系统不繁忙的状态下，swpd、so 的值不会持续很高，经常为 0。如果 swpd 过高，那么就是系统内存经常不够用。
- 对 CPU 监控，我们可以查看 r（运行进程数）、us、sy、id（CPU 空闲），如果 r 的数字大于系统 CPU 个数，则面临 CPU 不够用的危险，通过 id 分析，如果过小，则可以判断是 CPU 不足。

#### iostat

- `iostat -x` 一般情况下，%util 应该越小越好，10%以下正常，30%IO 比较繁忙。50%以上一般是有问题的

#### top

- 1 按 CPU 核数查看
- P
- M
- c 查看完整进程命令
- top -Hp pid 查看线程数

#### ps

- `ps -eo “pid,cmd,lstart” | grep pid` 查看进程启动时间
- `ps -ef f` 查看最近进程（常用）

#### find

- `find . -type f -mtime +3` 修改时间大于 3 天的文件
- `find . -type f -mtime +3 | xargs rm -rf` 查找并删除

#### du

- `du -sk * | sort -n | cut -f2 | xargs -d '\n' du -sh` 按文件大小排序显示
- `du -hs` 常用

#### awk

`grep 'update_profile.*Android' access-20180131.log |awk -F 'POST' '{print $2}'|awk -F '&' '{print $26}'|awk -F ' ' '{print $1}'|awk -F '=' '{print $2}'|sort -n|uniq -c|sort -nr|head -100`

-F 以空格分割

#### xargs

```bash
# xargs 使用
//查询redis members里面的值
redis-cli smembers myset|awk '{print $1}'|xargs -I {} redis-cli get {}
```

#### 查看进程监听的端口

```bash

查看程序对应进程号：ps –ef|grep 进程名

REDHAT :查看进程号所占用的端口号：netstat –nltp|grep 进程号

ubuntu:查看进程占用端口号：netstat -anp|grep pid

Linux下查看端口号所使用的进程号：

使用lsof命令： lsof –i:端口号

```

#### 其他

```bash
vim 块注释
ctrl+v
选择
大写I
输入注释符号
ESC 等一秒

抓包
tcpdump -i lo0 port 1234 -A
```
