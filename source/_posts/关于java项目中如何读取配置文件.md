---
title: 关于java项目中如何读取配置文件
date: 2016-07-22 09:27:32
tags: [java, spring, spring mvc, 读取配置文件, property file]
---

# 关于java项目中如何读取配置文件

## - Using multiple property files (via PropertyPlaceholderConfigurer) in multiple projects/modules
If you ensure that every place holder, in each of the contexts involved, is ignoring unresolvable keys then both of these approaches work. For example:
```
<context:property-placeholder
location="classpath:dao.properties,
          classpath:services.properties,
          classpath:user.properties"
ignore-unresolvable="true"/>
```
or
```
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:dao.properties</value>
                <value>classpath:services.properties</value>
                <value>classpath:user.properties</value>
            </list>
        </property> 
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
    </bean>
```

## - Spring MVC : read file from src/main/resources
```
Resource resource = new ClassPathResource(fileLocationInClasspath);
InputStream resourceInputStream = resource.getInputStream();
```

## - How do I load a resource and use its contents as a string in Spring
```
<bean id="contents" class="org.apache.commons.io.IOUtils" factory-method="toString">
    <constructor-arg value="classpath:path/to/resource.txt" type="java.io.InputStream" />
</bean>
```
This solution requires Apache Commons IO.

Another solution, suggested by @Parvez, without Apache Commons IO dependency is
```
<bean id="contents" class="java.lang.String">
    <constructor-arg>
        <bean class="org.springframework.util.FileCopyUtils" factory-method="copyToByteArray">
            <constructor-arg value="classpath:path/to/resource.txt" type="java.io.InputStream" />
        </bean>     
    </constructor-arg>
</bean>
```