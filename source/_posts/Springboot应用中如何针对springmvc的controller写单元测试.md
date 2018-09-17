---
title: Springboot应用中如何针对springmvc的controller写单元测试
date: 2016-07-23 12:54:56
tags: [spring boot, java, junit, 测试， spring mvc, Springboot应用中如何针对springmvc的controller写单元测试]
categories: java-spring
---


## An example test for your controller can be something as simple as
```
public class DemoApplicationTests {

    final String BASE_URL = "http://localhost:8080/";
    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = standaloneSetup(new HelloWorld()).build();
    }

    @Test
    public void testSayHelloWorld() throws Exception {
        this.mockMvc.perform(get("/").accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
                .andExpect(status().isOk())
                .andExpect(content().contentType("application/json"));

    }
}
```



## The new testing improvements that debuted in Spring Boot 1.4.M2 can help reduce the amount of code you need to write situation such as these.

The test would look like so:
```
@RunWith(SpringRunner.class)
@WebMvcTest(HelloWorld.class)
public class UserVehicleControllerTests {

    @Autowired
    private MockMvc mvc;

    @Test
    public void testSayHelloWorld() throws Exception {
        this.mockMvc.perform(get("/").accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
                .andExpect(status().isOk())
                .andExpect(content().contentType("application/json"));

    }

}
```



## 还有一种方式是使用TestRestTemplate

```
package controller;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.boot.test.TestRestTemplate;
import org.springframework.boot.test.WebIntegrationTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.web.client.RestTemplate;

import ipicture.service.post.AppServicePost;
import ipicture.service.post.model.JsonObject;
import ipicture.service.post.model.Subject;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = AppServicePost.class)
@WebIntegrationTest("server.port:8083")// 使用0表示端口号随机，也可以具体指定如8888这样的固定端口
public class SubjectControllerTest {

    private RestTemplate template = new TestRestTemplate();
    @Value("${server.port}")// 注入端口号
    private int port;
    
    private String getBaseUrl() {
        return "http://localhost:" + port;
    }
    
    @Test 
    public void test() {
        Subject s = new Subject();
        s.setCreator(1l);
        s.setCreated(new Date());
        s.setSubjectName("test5");
        s.setDescr("test subject");
        s.setDeleted(0);
        s.setParent_id(0);
        s.setStatus(0);
        s.setType(0);
        String url = getBaseUrl() + "/subject/save";
        String result = template.postForObject(url, s, String.class);
    }
}

```