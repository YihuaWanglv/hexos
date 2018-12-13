---
title: '[elasticsearch]windows下安装elasticsearch5-6'
date: 2018-12-13 17:56:28
tags: [elasticsearch, windows]
categories: elasticsearch
---

elasticsearch>5后，elasticsearch-head

不能放在elasticsearch的 plugins、modules 目录下

不能使用 elasticsearch-plugin install

直接启动elasticsearch即可

安装 elasticsearch-head

修改 elasticsearch/config/elasticsearch.yml

添加
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

下载 elasticsearch-head 或者 git clone 到随便一个文件夹

安装nodejs
```
cd /path/to/elasticsearch-head

npm install -g grunt-cli

npm install

grunt server
```
http://localhost:9100/

Enjoy it.