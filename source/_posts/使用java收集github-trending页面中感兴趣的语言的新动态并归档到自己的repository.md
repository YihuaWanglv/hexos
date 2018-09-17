---
title: 使用java收集github-trending页面中感兴趣的语言的新动态并归档到自己的repository
date: 2017-03-10 22:00:02
tags: [java,spring boot,github-trending,Runtime,github-repository]
categories: project
---


# java-github-trending

## what is this?
[github-trending](https://github.com/trending) 展示了每天趋势较好的一些github项目。
java-github-trending项目的目的就是每天定时拉取github-trending页面中感兴趣的语言的流行项目，并以日期为分组归档到markdown文件里面，最后提交到github的repository。

- 我的项目github地址：[github-trending](https://github.com/YihuaWanglv/github-trending)

## how to use?

- @Before
首先要有一个github账号，然后要准备一个运行应用的环境.

### 在你的运行环境中生成一个ssh key， 然后保存到github的key列表中.

### git clone project
```
git clone git@github.com:YihuaWanglv/github-trending.git
```

### maven package
```
cd github-trending
mvn clean package
```

### move github-trending.jar to project root path

### 将jar和sh脚本设置为可执行，然后nohu  run github-trending.jar
```
chmod +x rungit.sh
chmon +x github-trending.jar
nohup java -jar github-trending.jar < /dev/null > /data/logs/github-trending.log 2>&1 &
```

## how it works?

using spring @Scheduled as daily task.
using Jsoup lib to get and parse page content
use java.io lib to create markdown file and append content to md file.
use java Runtime class to run a shell to commit and push to github.

tips: 这个功能本身使用简单的java application，再配合linux的cron定时执行shell，即可更简单的做到，但由于一些原因，我使用spring boot来构建这个程序。
