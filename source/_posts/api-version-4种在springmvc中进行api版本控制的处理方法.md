---
title: '[api_version]4种在springmvc中进行api版本控制的处理方法'
date: 2016-06-28 15:31:54
tags: [java, api_verion, spring mvc]
categories: version
---

https://github.com/mindhaq/restapi-versioning-spring.git
https://github.com/augusto/restVersioning.git

这两个git项目里提供了4种在springmvc中进行api版本控制的处理方法，你们看怎么样，能否不使用冗余部署的方式。

1. /api/v1/xxx
controller中的方法注解写法：
```
    @ResponseBody
    @RequestMapping(value = "/apiurl/{version}/hello", method = GET, produces = APPLICATION_JSON_VALUE)
    public Hello sayHelloWorldUrl(@PathVariable final ValidVersion version) {
        return new Hello();
    }
```
2. /api/xxx
在header中添加"X-API-Version":"v1"来进行版本请求的区别
controller中的方法注解写法：
```
    @ResponseBody
    @RequestMapping(value = "/apiheader/hello", method = GET, produces = APPLICATION_JSON_VALUE)
    public Hello sayHelloWorldHeader(@RequestHeader("X-API-Version") final ValidVersion version) {
        return new Hello();
    }
```

3. /api/xxx
在header中添加Accept: "application/vnd.company.app-v1+json"来进行版本区别
controller中的方法注解写法：
```
    @ResponseBody
    @RequestMapping(
        value = "/apiaccept/hello", method = GET,
        produces = {"application/vnd.company.app-v1+json", "application/vnd.company.app-v2+json"}
    )
```
4. /api/xxx
在header中添加Accept: "application/vnd.company.app-v1+json"来进行版本区别
controller写法：
```
@Controller
@VersionedResource(media = "application/vnd.app.resource")
public class TestController {

    @RequestMapping(value = {"/resource"}, method = RequestMethod.GET)
    @VersionedResource(from = "1.0", to = "1.0")
    @ResponseBody
    public Resource getResource_v1() {
        return new Resource("1.0");
    }

    @RequestMapping(value = {"/resource"}, method = RequestMethod.GET)
    @VersionedResource(from = "2.0")
    @ResponseBody
    public Resource getResource_v2_onwards() {
        return new Resource("2.0");
    }
}