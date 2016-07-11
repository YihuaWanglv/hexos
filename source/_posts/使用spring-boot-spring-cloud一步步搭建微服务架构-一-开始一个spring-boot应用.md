---
title: '[使用spring-boot+spring-cloud一步步搭建微服务架构]一-开始一个spring-boot应用'
date: 2016-07-11 20:24:25
tags: [java, spring boot, spring cloud, 微服务, 使用spring-boot+spring-cloud一步步搭建微服务架构]
---

## 一. 目标愿景

**我们的目标是：使用spring  boot + spring cloud一步步搭建微服务架构**
本系列demo项目githu地址：
https://github.com/YihuaWanglv/lannisi

### 1. 都说服务化，以前我们是怎么搭建soa服务的？之前的架构有什么未达理想？

之前，一个比较流行的soa服务化方案是dubbo/dubbox+zookeeper，配上spring boot。
这个方案是比较流行的方案之一，但是有几点未达理想的地方：
- 1) dubbo的性能并不如一些其他服务化开发工具，比如thrift和ice。
- 2) dubbo是一个RPC的半完善解决方案，中规中矩，配套的软件基础设施不全。问题定位、熔断和监控方面的问题让人没有那么的放心。
- 3) 服务提供者和消费者都需要使用xml文件配置，不优雅。

### 2. 为什么使用Spring Cloud ？

Spring Cloud 提供了一整套分布式服务开发的工具，带着一些问题，我们希望在spring could中寻找答案。
Spring Cloud有什么特点？
- 1) 集成了springboot与docker，用起来很方便。配置中心，服务中心，服务，客户端等等组件开启，都非常的简单，往往只需几行代码。
- 2) 提供了一整套分布式服务开发的工具，从边缘服务的Zuul，到服务发现Eureka ，再到hystrix 熔断机制，是一套完整的生态。特别是hystrix，提供了完整的熔断机制，可以很轻易的引入现有系统。
- 3) Spring Cloud有一个不好的地方是，实际编码会有侵入性。

### 3. 为什么使用spring boot？

- 1) spring boot有pivotal和netfix背书，是一套完整的企业级应用的开发方案，天然集成分布式云架构spring-cloud。
- 2) spring-boot的完全抛弃以往java项目配置文件过多的“陋习”，开启一个项目只需几行代码。
- 3) Sprinboot允许项目使用内嵌的tomcat像启动普通java程序一样启动一个web项目.由于有了这个特性，项目不再需要打war包部署到tomcat，而是打成jar包，直接使用java -jar命令启动.


## 二. 开始第一个springboot项目
在一切开始之前，我们首先要知道如何开始一个springboot项目。

### 1. 创建一个空的maven项目，pom.xml中添加maven依赖
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 2. 编写项目启动入口App.java
```
@SpringBootApplication
public class App 
{
    public static void main(String[] args) throws Exception {
        SpringApplication.run(App.class, args);
    }
}
```

ok! done!
这样就已经能直接使用spring boot了.
启动App.java，spring boot就会使用内置的tomcat直接在本机的8080端口开启一个服务。

### 3. 再进一步，为应用引入spring mvc
```
@Controller
public class SampleController {

    
    @RequestMapping("/")
    @ResponseBody
    String home() {
        String data = "";
        return "Hello World!";
    }
}
```


启动App.java，访问localhost:8080, 即可遇见“Hello World!”

helloword项目也可参照：https://github.com/YihuaWanglv/lannisi 下的boot-sample-helloword项目。
https://github.com/YihuaWanglv/lannisi/tree/master/boot-sample-helloword