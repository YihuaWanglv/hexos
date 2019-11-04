---
title: springboot-不同环境的编译打包运行
date: 2019-11-04 15:20:13
tags: [spring boot,profiles,编译打包]
categories: java-spring
---



## 不同环境的编译打包运行

### 项目一共4套环境

- local：本地开发的环境
- dev：开发环境
- test：测试环境
- prod：正式环境

### 不同环境的打包

使用`mvn clean package -P${profile.active}`命令对不同环境进行打包

profile.active变量取值：
- local
- dev
- test
- prod

例如：要对测试环境打包，则使用命令`mvn clean package -Ptest`

这样application.yml文件中的`spring.profiles.active`取值就是：test，从而使用`application-test.yml`文件


### 运行

本地IDE直接运行BacApplication.java

开发和测试环境打包后，使用`java -jar`运行即可，其他参数可自行定制，无需再指定环境

- 对于正式环境，在jar包上传到线上服务器后，可以通过运行参数选择存放于服务器上的特定配置文件

```shell script
java -jar target/bac.jar --spring.config.location=file://${file.path}
```

变量file.path为配置文件路径


例如：

- linux

```shell script
java -jar target/bac.jar --spring.config.location=file:///data/bac/config/application-prod.yml
```

- windows

```shell script
java -jar target/bac.jar --spring.config.location=file:///D:/bac/application-local.yml
```



```java
spring:
  profiles:
    active: @profile.active@
```


```java
<profiles>
        <profile>
            <id>local</id>
            <properties>
                <profile.active>local</profile.active>
            </properties>
        </profile>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <profile.active>dev</profile.active>
            </properties>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profile.active>test</profile.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profile.active>prod</profile.active>
            </properties>
        </profile>
    </profiles>
```
