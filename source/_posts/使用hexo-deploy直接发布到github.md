---
title: 使用hexo-deploy直接发布到github
date: 2015-12-31 17:08:52
tags: [hexo,deploy]
categories: hexo
---

本文说明如何使用hexo deploy直接提交到github发布文章。

### 1.npm安装需要的东西
$ npm install hexo-deployer-git --save
$ npm install hexo-deployer-heroku --save
$ npm install hexo-deployer-rsync --save
$ npm install hexo-deployer-openshift --save
$ npm install hexo-deployer-ftpsync --save


### 2.修改配置文件_config.yml
deploy:
  type: git
  repository: https://github.com/YihuaWanglv/yihuawanglv.github.io.git
  branch: master

### 3.执行命令提交发布
$ hexo clean
$ hexo generate
$ hexo deploy

over.


### 参考：https://hexo.io/zh-cn/docs/deployment.html
