---
title: '[springboot]在spring项目中连接数据库以及spring-datajpa和mybatis的使用'
date: 2016-04-09 11:37:07
tags: [spring boot,mysql,数据库,spring data jpa,mybatis]
categories: java-spring
---

## 1.在项目中添加数据库配置（添加数据源等配置）
spring boot推崇约定大于配置，即是说，即使你什么也配置，spring boot也会为你添加一些约定俗成的默认配置。
比如数据源dataSource就是，如果你的spring boot项目没有数据源配置，那么默认情况下项目启动会失败，提示你配置数据源。
如果你的项目真的不需要数据源，没有数据库url等配置，那么你需要显式的使用注解配置告诉spring boot。
在启动类添加注解：
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class, HibernateJpaAutoConfiguration.class}

回到数据源配置上来。
首先,在配置文件application.properties里面配置数据库url
```
spring.datasource.url=jdbc:mysql://localhost/dbname
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
使用mysql数据库，还需要添加maven依赖
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
然后，添加数据库配置类

```
import javax.sql.DataSource;
import org.apache.ibatis.session.ExecutorType;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

@Configuration
public class DatabaseConfig {
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        return sessionFactory.getObject();
    }
}
```
如此，项目中就有了数据源


## 2.在项目中使用spring-data-jpa
添加maven依赖：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
添加Repository类：
```
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.repository.CrudRepository;

import com.xxx.xxx.model.User;

public interface UserRepository extends CrudRepository<User, Long> {

    Page<User> findAll(Pageable pageable);
    Page<User> findByNameContainingAndTypeContainingAllIgnoringCase(String name, Integer type, Pageable pageable);
    User findByNameAndTypeAllIgnoringCase(String name, Integer type);
    User findByName(String name);
}
```
pring-data-jpa的使用极其简单，只需要配置好对应的Repository接口即可，不需要操作的具体实现
这个例子中，UserRepository继承了CrudRepository，默认即拥有基础的增删改查接口。
然后你只需添加一些你需要的额外接口方法。
同样你只需按照其约定的方式写好接口方法，而不需要具体实现。

使用：
在需要的地方注解注入即可。
@Autowrite UserRepository userRepository;


## 3.在项目中使用mybatis
添加maven依赖：
```
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.3</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
</dependency>
```

mybatis的使用，和jpa一样，默认也是只需要定义Mapper接口即可，不需要实现类，但需要具体实现的sql。
你可以使用xml写sql，或者在接口上面使用注解注入sql两种方式。

注解sql例子：
```
public interface ItemMapper {
    @Select("select * from item")
    List<Item> findAll();
}
```
而如果是xml配置文件的话，则需要mapper xml中的sql id和Mapper接口中的接口方法名称一致。

当然，如果习惯了使用具体实现，使用SqlSessionTemplate，也可以实现。
首先需要改一下DatabaseConfig配置类：
```
import javax.sql.DataSource;
import org.apache.ibatis.session.ExecutorType;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

@Configuration
@MapperScan(basePackages="com.xxx.xxx.mapper")
public class DatabaseConfig {
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setConfigLocation(new ClassPathResource("mybatis-config.xml"));
        return sessionFactory.getObject();
    }
    @Bean
    @ConditionalOnMissingBean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory, ExecutorType.SIMPLE);
    }
}
```
添加mybatis配置文件：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.xxx.xxx.model"/>
    </typeAliases>
    <mappers>
        <mapper resource="mapper/xxxMapper.xml"/>
    </mappers>
</configuration>
```

具体Mapper实现类：
```
@Component
public class ItemMapper {
    @Autowired private SqlSessionTemplate sqlSessionTemplate;

    public Item selectItemById(long id) {
        return this.sqlSessionTemplate.selectOne("selectItemById", id);
    }
}
```

done！