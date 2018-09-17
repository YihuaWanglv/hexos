---
title: '[springboot][items][项目使用实践]springboot get started'
date: 2016-02-01 17:10:18
tags: [springboot,itime]
categories: java-spring
---

往后将通过一个时间记录web app项目，实践并记录spring boot的使用
为了快速实现，以及以后能更灵活的扩展，后台选用spring boot微服务框架。

### 1.pom.xml中添加maven依赖
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

### 2.编写项目启动入口App.java
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

### 再进一步，为应用引入spring mvc
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

