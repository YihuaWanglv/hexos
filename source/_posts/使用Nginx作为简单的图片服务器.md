---
title: 使用Nginx作为简单的图片服务器
date: 2017-03-10 23:59:19
tags: [ngnix, picture, server]
categories: deploy
---

## 说明：
- 为什么要用Nginx代理静态图片？

图片服务器是我们经常要用到的，在开始的时候，当还没用上阿里云oss，并且觉得FastDFS略微重量级的时候，就可以使用Nginx作为简单的图片服务器。

一下就是Nginx的conf配置文件例子：
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       80;
        server_name  localhost;
        location /images {
            root   /data/icms;
            add_header 'Access-Control-Allow-Origin' *;
            index  index.html index.htm;
            charset utf-8;
        }
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```

- 所有http://domain/image/xxx.png 的图片都会请求到服务器/data/icms/images内对应的图片。
