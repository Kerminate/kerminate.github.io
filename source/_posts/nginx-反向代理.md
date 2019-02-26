---
title: nginx 反向代理
date: 2018-11-17 12:35:39
tags: 工具
categories: 工具
---

当只有一台服务器时，而服务器上有多个 node 的服务，我们需要使用 nginx 进行反向代理，使每个服务能够正常被访问到。

<!--more-->

## 安装 nginx
```
yum install nginx
```

## 配置 nginx
安装完后，需要修改配置文件
```
cd /etc/nginx/conf.d  // 进入配置文件
vim app1.conf  // 编写 app1 服务的配置文件
```

将以下内容写入该文件
```shell
upstream app1 {
    server 127.0.0.1:7788;
    keepalive: 64;
}
server {
    listen 80;
    server_name app1.kerminate.me;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forward_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_pass https://app1;
    }
}
```

## 重启 nginx 服务
```
service nginx reload
```