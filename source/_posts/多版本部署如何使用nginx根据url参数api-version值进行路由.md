---
title: 多版本部署如何使用nginx根据url参数api_version值进行路由
date: 2016-07-26 18:53:07
tags: [冗余部署, nginx, api_version, 路由]
categories: deploy
---




## 目标：实现多版本部署，并且使用nginx根据HTTP请求中的url参数'api_version'来进行请求的路由，将对应版本的请求分发到目标版本实例当中。


## 实际测试场景：
tomcat部署启动了2个实例：
- 127.0.0.1:8080
- 127.0.0.1:8081

本地部署一个nginx服务器，对所有请求进行代理和路由。
部署的nginx简单配置如下：
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
        location /client {
            if ( $arg_api_version = '1.0.0' ){
                proxy_pass http://localhost:8080;
                break;
            }
            if ( $arg_api_version = '1.0.1' ){
                proxy_pass http://localhost:8081;
                break;
            }
            proxy_pass http://localhost:8080;
        }
        location / {
             proxy_pass http://localhost:8080;
        }
    }
}
```

预先配置好本地host，以便直接使用域名访问。
```
127.0.0.1 local.xxx.com
```

## 说明：
- 1.“listen       80;”配置nginx监听80端口，接收所有请求。
- 2.“location /client { ... }”部分将会匹配到所有以“/client”开头的请求
- 3.“location /client { ... }”部分，通过$http_api_version获取到url中字段名为'api_version'的参数值，用if判断版本值，做相应的路由。最后如果都没有匹配到，就会匹配最后一行“proxy_pass http://localhost:8080;”走8080实例。
- 4.“location / { ... }” 部分将会匹配其余的所有请求，路由到“http://localhost:8080”。这样，除“/client”前缀开头的请求外，都会默认匹配到一个固定的实例。别的请求都不会受到多版本路由配置的影响。


