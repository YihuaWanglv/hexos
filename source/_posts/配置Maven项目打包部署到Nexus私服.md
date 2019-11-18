---
title: 配置Maven项目打包部署到Nexus私服
date: 2019-11-18 11:36:26
tags: [java, maven, Nexus, 部署, 私服]
categories: java
---




# 配置Maven项目打包部署到Nexus私服


要点总结：

- 对于公共二方库的开发人员，需要配置maven/conf下setting.xml配置,增加<servers>节点。项目中已添加私服仓库地址和部署插件，开发人员不需要再配置。
- 对于使用公共二方库的项目，需要在maven配置文件中的mirrors添加<mirror>镜像。也可以在项目的pom文件中添加仓库，然后在pom文件中添加需要使用的库的<dependency>即可。


# 1. maven/conf下setting.xml配置,增加<servers>节点

增加<servers>节点
```xml
<servers>
    <server>
        <id>maven-public</id>
        <username>dev</username>
        <password>devmvn.db</password>
     </server>
     <server>
        <id>maven-snapshots</id>
        <username>dev</username>
        <password>devmvn.db</password>
     </server>
     <server>
        <id>maven-releases</id>
        <username>dev</username>
        <password>devmvn.db</password>
     </server>
</servers>

```

# 2. 二方库项目中pom.xml配置，增加distributionManagement节点，注意与properties节点平级。

```xml
<distributionManagement>
    <repository>
        <id>maven-releases</id>
        <name>maven-releases</name>
        <url>http://ip:port/repository/maven-releases/</url>
        <layout>default</layout>
    </repository>
    <snapshotRepository>
        <id>maven-snapshots</id>
        <name>maven-snapshots</name>
        <url>http://ip:port/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```


- 需要打包部署到私服的工程，则执行maven命令`mvn clean deploy`即可


# 3. 需要使用依赖的二方库的工程，则需要在maven的conf中配置mirrors节点，或者在工程的pom文件中配置好私服仓库地址

- 方式1：mirrors:

```xml
<mirror>     
    <id>bolingcavalry-nexus-releases</id>     
    <mirrorOf>*</mirrorOf>     
    <url>http://ip:port/repository/maven-public/</url>     
</mirror>    
```

- 方式2：在工程的pom文件中配置私服仓库地址

```xml
<repositories>
    <repository>
        <id>maven-public</id>
        <name>maven-public</name>
        <url>http://ip:port/repository/maven-public/</url>
    </repository>
</repositories>
```


# 4. 依赖准备

1. 在Nexus服务器上创建一个能deploy的Nexus账号用户名和密码
2. 设置账号为允许上传release的jar包
3. 设置账号为允许上传snapshots的jar包

# 5. 备选配置记录

部署到nexus，除了在pom中添加部署插件，也可以以设置maven config文件的形式设置profile，配置方法记录如下：

## 5.1. 方式1

### 5.1.1 设置私服地址，在<profiles>节点之下增加<profile>

```xml
<profile>
    <id>test</id>
    <repositories>
        <repository>
            <id>test</id>
            <name>test</name>
            <!-- 该 url 没有意义，可以随便写，但必须有。 -->  
            <url>http://*</url>
            <releases><enabled>true</enabled></releases>  
            <snapshots><enabled>true</enabled></snapshots>  
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>test</id>
            <name>local private nexus</name>
            <url>http://ip:port/repository/maven-public/</url>
            <releases><enabled>true</enabled></releases>  
            <snapshots><enabled>true</enabled></snapshots>  
         </pluginRepository>  
    </pluginRepositories>  
</profile>
```

### 5.1.2 激活配置

```xml
<activeProfiles>
    <activeProfile>test</activeProfile>
</activeProfiles>
```


## 5.2. 方式2

```xml
<repository>
    <id>maven-releases</id>
    <url>http://ip:port/repository/maven-releases/</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>true</enabled></snapshots>
</repository>
<repository>
    <id>maven-snapshots</id>
    <url>http://ip:port/repository/maven-snapshots/</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>true</enabled></snapshots>
</repository>


<pluginRepository>
    <id>maven-releases</id>
    <url>http://ip:port/repository/maven-releases/</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>true</enabled></snapshots>
</pluginRepository>
<pluginRepository>
    <id>maven-snapshots</id>
    <url>http://ip:port/repository/maven-snapshots/</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>true</enabled></snapshots>
</pluginRepository>
```


