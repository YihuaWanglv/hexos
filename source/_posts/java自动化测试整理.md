---
title: java自动化测试整理
date: 2016-08-12 13:14:27
tags: [Test, 测试, 单元测试, 自动化测试]
categories: java
---



## java自动化测试整理



## 1. 自动化测试的4个关键部分
![](/images/test-001.png)

- 使用svn/git作为代码版本管理
- 使用maven构建项目，依赖和目录统一
- 使用junit/testng+mockito写单元测试
- 使用jenkins执行项目构建和测试任务
- 使用Sonar作为自动化单元测试反馈报告统一展现平台


## 2. web自动化测试管理工具
- 使用 Selenium 实现基于 Web 的自动化测试
- 使用 Rest-Assured 测试 REST API
- api测试，从手工到平台的演变：运用web+httpclient实现，测试的可配置


## 3. restapi的自动化测试之路
- Rest-Assured或其他工具，编写针对api的单元测试；
- 新代码提交，自动触发构建，成功部署服务器；
- 自动运行使用 Rest-Assured 编写的单元测试，对 REST API 进行测试


## 4. 自动化测试工具脑图
![](/images/test-002.png)