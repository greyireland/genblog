---
title: docker命令
tags:
  - tags
categories:
  - categories
date: 2019-02-21 20:40:22
---

# Docker使用：






## 常用命令

### 容器生命周期管理

- [run](http://www.runoob.com/docker/docker-run-command.html)
- [start/stop/restart](http://www.runoob.com/docker/docker-start-stop-restart-command.html)
- [kill](http://www.runoob.com/docker/docker-kill-command.html) 向容器发送信号
- [rm](http://www.runoob.com/docker/docker-rm-command.html) 常见combo是：stop x;rm x ;build x;run x
- [pause/unpause](http://www.runoob.com/docker/docker-pause-unpause-command.html)
- [create](http://www.runoob.com/docker/docker-create-command.html)
- [exec](http://www.runoob.com/docker/docker-exec-command.html) 在运行的容器中执行命令

### 容器操作

- [ps](http://www.runoob.com/docker/docker-ps-command.html) 显示容器
- [inspect](http://www.runoob.com/docker/docker-inspect-command.html) 获取容器/镜像的元数据。返回json格式的数据
- [top](http://www.runoob.com/docker/docker-top-command.html) 查看容器中的进程信息
- [attach](http://www.runoob.com/docker/docker-attach-command.html) **:**连接到正在运行中的容器。
- [events](http://www.runoob.com/docker/docker-events-command.html) docker相关的事件
- [logs](http://www.runoob.com/docker/docker-logs-command.html) 应该是打印到stdout上的日志吧！！！
- [wait](http://www.runoob.com/docker/docker-wait-command.html)
- [export](http://www.runoob.com/docker/docker-export-command.html)
- [port](http://www.runoob.com/docker/docker-port-command.html) 显示容器的端口映射

### 容器rootfs命令

- [commit](http://www.runoob.com/docker/docker-commit-command.html) 类似git commit 一样修改之后保存新版本docker commit -m "commit test" uuu fuiboom/myubuntu:v1 给运行的容器创建一个新的保存镜像（一般常用Dockerfile和build来构建镜像）
- [cp](http://www.runoob.com/docker/docker-cp-command.html) 容器与主机的数据拷贝
- [diff](http://www.runoob.com/docker/docker-diff-command.html) 显示容器文件结构的改变（显示自己改了什么东西）

### 镜像仓库

- [login](http://www.runoob.com/docker/docker-login-command.html)
- [pull](http://www.runoob.com/docker/docker-pull-command.html) 拉取别人已经配置好的环境（非常好用的命令）
- [push](http://www.runoob.com/docker/docker-push-command.html) push自己的镜像，（需要登录）
- [search](http://www.runoob.com/docker/docker-search-command.html)

### 本地镜像管理

- [images](http://www.runoob.com/docker/docker-images-command.html) 
- [rmi](http://www.runoob.com/docker/docker-rmi-command.html) 移除本地image(不然占用本地磁盘空间)
- [tag](http://www.runoob.com/docker/docker-tag-command.html) 创建一个镜像的引用docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG] 

> docker tag mysql_client_test fuiboom/mysql_client_test 把官方的改造重新命名一下就可以push
>
> docker push fuiboom/mysql_client_test

- [build](http://www.runoob.com/docker/docker-build-command.html) 根据Dockerfile创建一个镜像
- [history](http://www.runoob.com/docker/docker-history-command.html) 显示构建容器的历史数据
- [save](http://www.runoob.com/docker/docker-save-command.html) save和load搭配
- [import](http://www.runoob.com/docker/docker-import-command.html) export和import搭配

### info|version

- [info](http://www.runoob.com/docker/docker-info-command.html)
- [version](http://www.runoob.com/docker/docker-version-command.html)

### docker run 创建一个新的容器并运行一个命令

### 语法

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

OPTIONS说明：

- **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- **-d:** 后台运行容器，并返回容器ID；
- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **--name="nginx-lb":** 为容器指定一个名称；
- **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
- **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
- **-h "mars":** 指定容器的hostname；
- **-e username="ritchie":** 设置环境变量；
- **--env-file=[]:** 从指定文件读入环境变量；
- **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
- **-m :**设置容器使用内存最大值；
- **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- **--link=[]:** 添加链接到另一个容器；
- **--expose=[]:** 开放一个端口或一组端口；
- -P 大P使用expose的端口映射到主机的随机端口（感觉不好，还是自己指定比较好）

### 实例

使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

```
docker run --name mynginx -d nginx:latest
```

使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。

```
docker run -P -d nginx:latest
```

使用镜像nginx:latest以后台模式启动一个容器,将容器的80端口映射到主机的80端口,主机的目录/data映射到容器的/data。

```
docker run -p 80:80 -v /data:/data -d nginx:latest
```

使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

```
runoob@runoob:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/# 
```



### docker exec ：在运行的容器中执行命令

### 语法

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

OPTIONS说明：

- **-d :**分离模式: 在后台运行
- **-i :**即使没有附加也保持STDIN 打开
- **-t :**分配一个伪终端

### 实例

在容器mynginx中以交互模式执行容器内/root/runoob.sh脚本 （用容器中的程序执行某段脚本）

```
runoob@runoob:~$ docker exec -it mynginx /bin/sh /root/runoob.sh
http://www.runoob.com/
```

在容器mynginx中开启一个交互模式的终端（启动容器的一个可交互bash窗口）

```
runoob@runoob:~$ docker exec -i -t  mynginx /bin/bash
root@b1a0703e41e7:/#
```

启动、停止所有容器

```
docker start $(docker ps -a -q) 
```





#### TODO问题？？？

1. 怎么设置容器的cpu，内存，磁盘，网络大小限制？
2. 设置容器镜像

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://a2a3a2e1.m.daocloud.io 
```

```
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/docker daemon|ExecStart=/usr/bin/docker daemon --registry-mirror=http://a2a3a2e1.m.daocloud.io|g" /etc/systemd/system/docker.service
sudo systemctl daemon-reload
sudo service docker restart
```



```
sudo sed -i "s|ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd –registry-mirror=https://pee6w651.mirror.aliyuncs.com|g" /etc/systemd/system/docker.service
```



常用镜像：

#### redis

```
//-d 后台运行
docker run --name some-redis -d redis

//--link A:B 连接过去(自动设置host) --rm 终端退出时自动清除容器内容(跟-d不同时用)
docker run -it --link some-redis:redis --rm redis redis-cli -h redis -p 6379

//自定义conf配置文件
docker run -v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis redis redis-server /usr/local/etc/redis/redis.conf

```



#### mysql

```
//创建mysql容器 -e 设置环境变量
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

//客户端连接
docker run -it --link some-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

//mysql的客户端
docker run -it --rm mysql mysql -hsome.mysql.host -usome-mysql-user -p

//mysql使用配置文件/my/custom/config-file.cnf ==> /etc/mysql/conf.d/config-file.cnf
docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

//指定外部存储目录
docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

//dump数据到宿主机
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
```

#### nginx

```
//静态页面
docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx

//配置文件
docker run --name my-custom-nginx-container -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx

//debug模式运行
docker run --name my-nginx -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx nginx-debug -g 'daemon off;'
```



```
docker run --name my_zookeeper -p 2181:2181 -d zookeeper:latest
docker run -it --rm --link my_zookeeper:zookeeper zookeeper zkCli.sh -server zookeeper
https://segmentfault.com/a/1190000006907443

10.111.100.235:2181,10.111.100.236:2181,10.111.100.237:2181,10.111.100.238:2181/user.base.global_id_gen/
```


# 编写dockerfile文件

```
#dockerfile文件

FROM debian:jessie

RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

2.在 `Dockerfile` 文件所在目录执行：

```
$ docker build -t nginx:v3 .
```

```
docker build [选项] <上下文路径/URL/->
```

如果在 `Dockerfile` 中这么写：

```
COPY ./package.json /app/

```

这并不是要复制执行 `docker build` 命令所在的目录下的 `package.json`，也不是复制 `Dockerfile` 所在目录下的 `package.json`，而是复制 **上下文（context）** 目录下的 `package.json`。

一般来说，应该会将 `Dockerfile` 置于一个空目录下，或者项目根目录下。

#### COPY命令

> 源路径必须使用相对当前目录的路径	

从上下文路径复制文件到 镜像中某个目录

`COPY <源路径>... <目标路径>`

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR`指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

#### CMD命令

```
CMD [ "sh", "-c", "echo $HOME" ]
```

#### ENV命名

```
ENV NODE_VERSION 7.2.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" 
#后面可以使用到
```

下列指令可以支持环境变量展开（可以用这个环境变量）： `ADD`、`COPY`、`ENV`、`EXPOSE`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`。

#### EXPOSE

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

#### WORKDIR 

> 下面ENTRYPOINT命令会在这个目录下面执行命令

```
WORKDIR <工作目录路径> #以后可以在某层有一个 相对路径
```

#### entrypoint 入口

```
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off"]
```



3.运行docker容器中的项目：

// -t 新的伪终端tty 

-i 交互输入保持打开 

-d后台运行，在使用 `-d` 参数时，容器启动后会进入后台

-p指定外内端口映射，**-P:**将容器内部使用的网络端口映射到我们使用的主机上。有expose就用expose的，没有就随机一个

-v指定容器外目录与容器内目录映射

-w /usr/src/myapp :指定容器的/usr/src/myapp目录为工作目录

-e username="ritchie":设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--expose=[]:开放一个端口或一组端口；

--name="nginx-lb": 为容器指定一个名称；

```
sudo docker run -t -i ubuntu:14.04 /bin/bash
```

```
sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

终止：docker stop



###  常见命令

```
docker ps -a //所有
docker ps -l //最近
docker ps -n 5//最近5个
docker ps //运行中
```

```
docker inspect xx
docker inspect -f '{{.NetworkSettings.IPAddress}}' mymysql
docker top xx
```

```
docker images
docker images ubutun_*
docker tag ubuntu:15.10 runoob/ubuntu:v3
docker build -t runoob/ubuntu:v1 . 
docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
docker import  my_ubuntu_v3.tar runoob/ubuntu:v4
```



```
alias cp='cp -i'
alias dex='docker exec -i -t'
alias di='docker images'
alias dip='docker inspect --format '\''{{ .NetworkSettings.IPAddress }}'\'''
alias dkd='docker run -d -P'
alias dki='docker run -i -t -P'
alias dl='docker ps -l -q'
alias dpa='docker ps -a'
alias dps='docker ps'
alias drmf='docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
```



