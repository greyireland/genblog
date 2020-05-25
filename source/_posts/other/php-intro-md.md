---
title: php_intro.md
tags:
  - php
  - php-fpm
categories:
  - php
date: 2017-11-09 14:03:26
---

## php 入门

php 简单入门

- 安装 php、php-fpm
- 安装 nginx
- 安装 mysql
- 安装 composer(包管理工具)

运行 php-fpm:`php-fpm --fpm-config /usr/local/etc/php-fpm.conf --prefix /usr/local/var`

入门 helloworld.php

```php
<?php
echo "Hello World!";
?>
```

运行：php helloworld.php

## php 概念

[简单教程](https://www.runoob.com/php/php-tutorial.html)

## php-nginx 配置

```nginx
events{}
http {
  server {
    listen 8080;
    server_name localhost;
    root         /Users/xiaodong/PhpstormProjects/;
    location ~ \.php$ {
        # 设置监听端口
        fastcgi_pass   127.0.0.1:9000;
        # 设置nginx的默认首页文件(上面已经设置过了，可以删除)
        fastcgi_index  index.php;
        # 设置脚本文件请求的路径
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        # 引入fastcgi的配置文件
        include        fastcgi_params;
    }
  }
}

```

## 其他项目

一些框架

- thinkphp
- yii
- Laravel

## 问题记录

[mac 遇到问题](https://app.yinxiang.com/shard/s9/nl/16492052/663ce6a0-0aac-4da8-8f60-4f464a790fad?title=mac%20%E5%90%AF%E5%8A%A8php-fpm%20-%20%E9%83%AD%E8%83%9C%E9%BE%99%E7%9A%84%E6%8A%80%E6%9C%AF%E5%8D%9A%E5%AE%A2%20-%20%E5%8D%9A%E5%AE%A2%E9%A2%91%E9%81%93%20-%20CSDN.NET)
