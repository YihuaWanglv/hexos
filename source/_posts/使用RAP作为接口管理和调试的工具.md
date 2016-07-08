---
title: 使用RAP作为接口管理和调试的工具
date: 2016-07-08 14:06:27
tags: [接口文档, RAP, mock]
---

## 一、为什么使用RAP？
- 接口定义和管理，传统的方式是使用word文档。但word文档管理接口文档有不少问题。
比如，word文档在相互传递的过程中，可能会导致彼此文档版本不一致；word文档难以结构化的集中维护各项目的文档，而且编写麻烦；word文档的接口定义功能单一，仅仅提供文档功能。

- 而是用RAP作为接口文档定义管理的工具，则为我们提供了以下功能：
1.可以按团队和项目结构化的管理接口文档。
2.提供统一方便的接口定义方式。
3.为前端提供mock数据，以便在后端没有开发好后端接口前，提前根据接口进行开发。
4.校验后端提供的接口的数据格式和字段是否符合要求。

**虽然RAP在自动化测试方面功能还有欠缺，仍不失为接口管理一个十分方便的工具。**



## 二、RAP安装部署
### 1.下载RAP
在RAP的github主页上下载最新版本war包

### 2.环境准备
需要tomcat和mysql

### 3.部署
RAP在tomcat中要使用ROOT的方式部署。
- 把下载好的war包放入tomcat的webapps目录下，然后删除tomcat中webapps目录下原先的ROOT文件夹，重命名war包为ROOT.war
- 在mysql中建立rap_db数据库，导入RAPgithub代码中提供的sql脚本初始化数据库
- 启动tomcat
- 访问http://localhost:8080/

## 三、RAP的使用
### 1.注册账号
RAP自带账号管理和登陆功能，使用前先注册账号，登陆即可

### 2.添加团队、项目产品线等。
- 创建团队
![](/images/rap-001.png)
- 创建产品线
![](/images/rap-002.png)
![](/images/rap-003.png)

### 3.分组和项目管理
![](/images/rap-004.png)
![](/images/rap-009.png)

### 4.添加模块和页面
![](/images/rap-005.png)

### 5.添加接口
![](/images/rap-006.png)

### 6.为返回值字段设置自定义的mock数据
![](/images/rap-007.png)
我们可以为不同的字段自定义不同的mock数据，已适应实际需要。

### 7.前端使用接口mock数据提前进行开发
![](/images/rap-008.png)
这个例子中的“项目根路径”：localhost:8080/mockjs/1
就是这个mock服务的模拟服务路径
前端访问：http://localhost:8080/mockjs/1/client/machine/activate?drive_id=sfd&activate_code=sdf&mac_address=sdf
就是访问我们模拟的接口地址: /client/machine/activate