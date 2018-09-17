---
title: '[git]git常用-把本地项目提交到github'
date: 2015-12-31 18:04:12
tags: [git]
categories: git
---

### 1.把本地项目提交到github
touch README.md //新建说明文件
git init //在当前项目目录中生成本地git管理,并建立一个隐藏.git目录
git add . //添加当前目录中的所有文件到索引
git commit -m "first commit" //提交到本地源码库，并附加提交注释
git remote add origin https://github.com/chape/test.git //添加到远程项目，别名为origin
git push -u origin master //把本地源码库push到github 别名为origin的远程项目中，确认提交


### 2.如果有error: failed to push some refs to 'https://github.com/YihuaWanglv/myhexo.git'
有如下几种解决方法：

1.使用强制push的方法：
$ git push -u origin master -f 
这样会使远程修改丢失，一般是不可取的，尤其是多人协作开发的时候。
2.push前先将远程repository修改pull下来
$ git pull origin master
$ git push -u origin master
3.若不想merge远程和本地修改，可以先创建新的分支：
$ git branch [name]
然后push
$ git push -u origin [name]


