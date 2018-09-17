---
title: hexo生成的静态文件如何更新到自己的服务器上
date: 2018-09-10 01:27:35
tags: [hexo, blog, 自动发布, 自动部署, 云服务]
categories: hexo
---

github pages为每个github用户提供了一个免费的静态空间。

而hexo是一款快速、简洁且高效的博客框架。

hexo + github pages实现一个静态博客，是非常流行的做法。这个方案是非常好的，但有2点不好，一个是博客不是自己的域名，第二个是github.io的访问速度不是很快。

于是，我们就会考虑说怎么样把hexo生成的静态博客网页文件部署到自己的服务器上呢？

hexo部署的内容，可以参考之前的文章：

[如何使用github-pages和hexo搭建简单blog](http://blog.iyihua.cn/2015/12/31/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8github-pages%E5%92%8Chexo%E6%90%AD%E5%BB%BA%E7%AE%80%E5%8D%95blog/)

[使用hexo-deploy直接发布到github](http://blog.iyihua.cn/2015/12/31/%E4%BD%BF%E7%94%A8hexo-deploy%E7%9B%B4%E6%8E%A5%E5%8F%91%E5%B8%83%E5%88%B0github/)

以下是我的方案：

前提是，你已经利用xxx.github.io项目里面的静态文件，检出副本到你的服务器上，并利用nginx静态代理实现博客的访问。

然后自动更新的思路就是，hexo deploy的时候会提交更新到xxx.github.io项目，利用webhooks，向自己写的一个http服务器发送一个post请求，http服务器接收到post请求，利用shelljs库执行shell脚本，进入检出的xxx.github.io项目服务器副本，并git pull。即可更新自己的博客。


# 1. 把你的xxx.github.io项目检出到云服务器上

# 2. 在你的xxx.github.io git项目仓库中建立一个webhooks

webhooks的配置是playload=｛url｝

url为web服务接收http post请求的url地址；

# 3. Express实现的http服务器接收到http请求后，利用node 的shelljs库，执行服务器的shell脚本


由于请求简单，我这里直接利用Express来作为http的服务。

- app.js例子如下：
```
const express = require('express')
const shell = require("shelljs");
const app = express()

app.post('/cmd', (req, res) => {
    shell.exec("/home/app/build/update-blog.sh");
    res.send('exec ok!')
})

app.listen(3000, () => console.log('Example app listening on port 3000!'))
```

然后在服务器上部署这个http项目

# 4. shell脚本内的过程就是，cd进入云服务器检出的xxx.github.io项目根目录，然后执行git pull更新。

- 脚本例子：
```
cd /您的服务器上xxx.github.io项目根目录
git pull
```

# 5. 于是每当hexo执行'hexo deoloy'命令向xxx.github.io推送最新的修改后，xxx.github.io的webhooks就会向http服务器发送一个http请求。从而实现blog的更新。
