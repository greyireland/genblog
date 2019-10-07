---
title: supervisor使用
tags:
  - linux
  - supervisor
categories:
  - linux
date: 2019-05-12 16:40:49
---

# linux进程守护

### 安装
```
pip install supervisor
supervisord -v
```

### 命令
```
1.配置路径
mkdir -p /etc/supervisor

2.完整配置示例
echo_supervisord_conf > /etc/supervisor/supervisord.conf

3.修改配置
vim /etc/supervisor/supervisord.conf
添加如下
[include]
files=/etc/supervisor/conf.d/*.conf

4.添加如下配置
mkdir -p /etc/supervisor/conf.d/
vim /etc/supervisor/conf.d/xmysql.conf

[program:myxmysql]
command=xmysql -h localhost -u username -p username -d music -r 0.0.0.0 ; 运行程序的命令
directory=/home/xiaodong/node-v10.15.3-linux-x64/bin/ ; 执行前要不要先cd到
目录去，一般不用
autorestart = true   ; 程序异常退出后自动重启
autostart=true       ; 随着supervisord的启动而启动
startretries = 10     ; 启动失败自动重试次数，默认是 3
numprocs=1           ; 启动几个进程
stderr_logfile=/var/log/myxmysql.err.log ; 错误日志文件
stdout_logfile=/var/log/myxmysql.out.log ; 输出日志文件
environment=A=B ; 进程环境变量
user=root ; 进程执行的用户身份
stopsignal=KILL ; 用来杀死进程的
stopsignal=INT

5.启动
supervisord -c /etc/supervisor/supervisord.conf
或者重新加载配置
supervisorctl reload
```

### 额外
```
把配置文件中 inet_http_server 前面的分号去掉 在Supervisor中 ;是注释的格式 默认是不开启web界面的

[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9001        ; (ip_address:port specifier, *:port for all iface)
username=user              ; (default is no username (open server))
password=123 
```