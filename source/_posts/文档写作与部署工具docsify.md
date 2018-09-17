---
title: 文档写作与部署工具docsify
date: 2018-03-14 18:01:27
tags: [doc, docs, markdown, docsify]
categories: tools
---

# docsify：一个文档展示与部署的工具

基于md文档的写作，构建一个好看的文档展示样式，并启动服务提供访问。

## 官网

https://docsify.js.org

## 快速开始

推荐安装 docsify-cli 工具，可以方便创建及本地预览文档网站。
```
npm i docsify-cli -g
```

## 初始化项目

如果想在项目的 ./docs 目录里写文档，直接通过 init 初始化项目。
```
docsify init ./docs
```

## 开始写文档

初始化成功后，可以看到 ./docs 目录下创建的几个文件
```
index.html 入口文件
README.md 会做为主页内容渲染
.nojekyll 用于阻止 GitHub Pages 会忽略掉下划线开头的文件
直接编辑 docs/README.md 就能更新网站内容，当然也可以写多个页面。
```

## 本地预览网站

运行一个本地服务器通过 docsify serve 可以方便的预览效果，而且提供 LiveReload 功能，可以让实时的预览。默认访问 http://localhost:3000 。
```
docsify serve docs
```

## 多页文档

如果需要创建多个页面，或者需要多级路由的网站，在 docsify 里也能很容易的实现。例如创建一个 guide.md 文件，那么对应的路由就是 /#/guide。

假设你的目录结构如下：
```
-| docs/
  -| README.md
  -| guide.md
  -| zh-cn/
    -| README.md
    -| guide.md
```

那么对应的访问页面将是
```
docs/README.md        => http://domain.com
docs/guide.md         => http://domain.com/guide
docs/zh-cn/README.md  => http://domain.com/zh-cn/
docs/zh-cn/guide.md   => http://domain.com/zh-cn/guide
```
