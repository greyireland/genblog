---
title: k8s入门
tags:
  - k8s
categories:
  - docker
date: 2019-02-22 10:22:20
---

## docker 虚拟化实现

隔离是怎么实现的？

- namespace 进程隔离，名称映射【障眼法】
- Cgroups 限制进程使用资源，限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等。此外还可以对进程进行优先级设置、审计、进程挂起和恢复。

### cgroup 隔离配置

配置相关文件

```bash
$ mount -t cgroup
cpuset on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (rw,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct)
blkio on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
memory on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
```

创建一个 cgroup 配置

```bash
root@ubuntu:/sys/fs/cgroup/cpu$ mkdir container
root@ubuntu:/sys/fs/cgroup/cpu$ ls container/
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks
```

执行脚本，进程号 226，top 查看 CPU 占用 100%

```bash
while : ; do : ; done &
[1] 226
```

因为默认配置，quota 为-1 没有限制，period 默认为 100ms(100000us)：

```bash
cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
-1
cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us
100000
```

向 container 组的 cfs_quota 写入 20000us

```bash
echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
```

意味着这个进程只是用 20%CPU,然后写入被限制的进程 PID，使用 top 查看 CPU 占用 20%了

```bash
echo 226 > /sys/fs/cgroup/cpu/container/tasks
```

docker 运行容器时可以指定参数

```bash
docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us
100000
cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us
20000
```

说明这个容器只能占用 20%的 CPU

### mount namespace

挂载新的路径作为容器的根路径 rootfs
核心流程

1. 弃用 linux namespace
2. 指定 cgroups 参数
3. 切换进程的根目录

## 问题

docker 和 KVM 虚拟机区别？

- docker 本身不占用资源，KVM 本身占用 200M 内存
- KVM 虚拟化拦截，性能有损耗，docker 直接使用宿主机的进程，性能更好
- docker 隔离不是很彻底，linux 内核中很多对象和资源是不能被 namespace 化，如时间 宿主机改了容器也会改

容器是一个单进程吗？

> 是，一个容器本质就是一个进程，用户的应用进程实际上就是容器里 PID 为 1 的进程，也是后续创建的进程的父进程。所以没办法在一个容器运行两个不同的应用。

解决容器类 top 查看到宿主机信息问题？

> top 是从/proc 下面获取数据的，所以把宿主机的 /var/lib/lxcfs/proc/文件挂载到容器的/proc/就可以通过 lxcfs 读取到对应容器的内存，CPU 等限制
