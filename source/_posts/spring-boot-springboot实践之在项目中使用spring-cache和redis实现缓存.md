---
title: '[spring-boot]springboot实践之在项目中使用spring-cache和redis实现缓存'
date: 2016-04-09 10:43:01
tags: [spring boot,spring cache,redis,注解]
categories: java-spring
---

## 1.首先需要准备一个redis服务端作为缓存
redis下载安装启动，比较简单，请google之。

## 2.项目依赖
使用spring boot和spring cache，需要springboot依赖
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
```
连接redis，需要spring data redis和jedis包
```
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
```

## 3.程序配置实现与使用
不同于spring使用xml配置文件来配置bean，spring boot使用java bean直接在java类中配置需要用到的Bean。

### 添加redis cache相关的config类，如下：
```
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;

@Configuration
@EnableCaching
public class RedisCacheConfig extends CachingConfigurerSupport {
    @Bean
    public JedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory redisConnectionFactory = new JedisConnectionFactory();
        redisConnectionFactory.setHostName("127.0.0.1");
        redisConnectionFactory.setPort(6379);
        return redisConnectionFactory;
    }
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory cf) {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<String, String>();
        redisTemplate.setConnectionFactory(cf);
        return redisTemplate;
    }
    @Bean(name = "redisCacheManager")
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        cacheManager.setDefaultExpiration(300);
        return cacheManager;
    }
}
```
@Configuration标记此java类为spring boot的配置类
@EnableCaching标记为项目启用缓存
@Bean，每个被此注解标记的方法都会配置一个bean，等同于xml配置中的<bean>
从以上代码可以看到，使用spring cache和redis缓存，我们需要一个JedisConnectionFactory，一个RedisTemplate和一个CacheManager。
其中JedisConnectionFactory配置了redis服务的ip和端口，为了方便这里直接写在了程序里，可以移出去放到配置文件中读取。

### 如何使用？
使用非常简单，只需要在我们想要缓存的方法前加一个@Cacheable注解即可
下面是一个例子：
```
    @Cacheable(value = "find")
    public List<Object> find(Long id) {
        ...
    }
    @CacheEvict(value = "find", allEntries = true)
    public Object saveProject(Object object) {
        ...
    }
    @CacheEvict(value = "find" , allEntries = true)
    public void deleteProject(Long id) {
        ...
    }
```
这里@Cacheable标记的方法在首次调用后，将会将读取到的内容缓存到redis中，以后每次只要缓存中存在该缓存，读取都会从缓存中读取。
但如果实际数据改变了怎么办？
我查看spring cache的api后，没有适用的动态更新缓存的方法，所以这里适用了一种不怎么优雅的解决方案：
每次会对模型数据改变的方法，都清除一次缓存。清除后的第一次，查询会读取数据库，之后再缓存。
@CacheEvict标记的方法，表示将会清除缓存，value属性中配置了“find”，表示清除key为“find”的缓存数据。

这个方案不是很优雅，就是用起来简单，如果想更优雅的方案，应该要自己再实现一套自己可控的缓存程序。