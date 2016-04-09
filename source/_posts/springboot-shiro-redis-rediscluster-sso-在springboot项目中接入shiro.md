---
title: '[springboot+shiro+redis+rediscluster+sso]在springboot项目中接入shiro'
date: 2016-04-09 12:52:53
tags: [spring boot,shiro,java]
---

在之前的文章中，分别介绍了springboot启动，在springboot中使用redis作为缓存，在springboot中使用jpa和mybatis，现在将开始一个更大的综合工程：一步步介绍在springboot中使用shiro+redis-cluster+cookie实现一个跨域单点登录的方案。

现在开始是第一篇：在springboot项目中接入shiro

## 引入shiro的maven依赖
```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.2.3</version>
</dependency>
```

## 程序具体实现

### 首先，需要一个shiro的配置文件，这里使用的是ymal配置。配置文件：application.yml
```
shiro:
  realm: com.xxx.xxx.config.security.MyRealm
  loginUrl: /view/sign-in.html
  successUrl: /item.html
  unauthorizedUrl: /forbidden.html
  filterChainDefinitions:
    "/login": anon
    "/static/**": anon
    "/bower_components/**": anon
    "/logout": logout
    "/**": authc
```
说明：
realm: com.xxx.xxx.config.security.MyRealm
Realm，在shiro中相当于数据源，shiro的其他核心组件需要获取用户和认证数据，就是从Realm获取。
为了获得更好的自定义功能，通常我们会自己实现一个Realm.
所以这里使用realm: com.xxx.xxx.config.security.MyRealm配置了一个自定义的Realm。
loginUrl，定义了需要认证用户时，跳转到的登录页面
successUrl，定义登录成功后跳转的页面。通常也可以在自己登录认证方法里redirect到需要的页面。
unauthorizedUrl，定义未认证时显示的页面。
filterChainDefinitions，定义哪些路径应该做何种过滤策略。
anon，logout，authc这些都是shiro默认实现的过滤器filter。
anon表示可以匿名访问的路径，authc表示需要登录认证的路径

过滤器链使用最先匹配返回策略，所以我们需要把不需要认证即可访问的路径放在前面。

### 下面是自己实现的自定义Realm：MyRealm
```
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import com.xxx.model.base.UserDTO;
import com.xxx.remote.base.UserRemote;

public class MyRealm extends AuthorizingRealm {
    @Autowired UserRemote userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        UserDTO user = (UserDTO) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        List<String> roles = userService.findByUserId(user.getId());
        info.addRoles(roles);
        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        String username = usernamePasswordToken.getUsername();
        UserDTO user = userService.findUserByName(username);
        if (null != user) {
            CredentialsInfoHolder cih = new CredentialsInfoHolder(user.getPassword(), user.getSalt());
            return new SimpleAuthenticationInfo(user, cih, getName());
        }
        return null;
    }
}
```
MyRealm集成AuthorizingRealm，需要重写AuthorizingRealm的两个方法：doGetAuthorizationInfo，doGetAuthenticationInfo。
AuthorizationInfo represents a single Subject's stored authorization data (roles, permissions, etc) used during authorization (access control) checks only. 

doGetAuthorizationInfo方法返回一个AuthorizationInfo，AuthorizationInfo对象是一个单一的Subject对象，存储着用户的授权数据，只用于授权检查时使用。
doGetAuthenticationInfo方法，针对给定的用户，获取对应用户的认证数据，提供给认证用户身份时使用。

例子中UserRemote userService是提供用户数据的具体service服务。

### 有了数据源realm和shiro配置文件，现在开始接入shiro配置到spring容器
#### springboot shiro配置类：ShiroAutoConfig.java
```
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;
import org.springframework.context.annotation.Import;

import com.xxx.xxx.config.security.MyRealm;

@Configuration
@EnableConfigurationProperties(ShiroProperties.class)
@Import(ShiroManager.class)
public class ShiroAutoConfig {
    @Autowired private ShiroProperties properties;

    @Bean(name = "realm")
    @DependsOn("lifecycleBeanPostProcessor")
    @ConditionalOnMissingBean
    public MyRealm realm() {
        Class<?> relmClass = properties.getRealm();
        MyRealm r = (MyRealm) BeanUtils.instantiate(relmClass);
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
        credentialsMatcher.setHashAlgorithmName(Sha256Hash.ALGORITHM_NAME);
        r.setCredentialsMatcher(credentialsMatcher);
        return r;
    }

    @Bean(name = "shiroFilter")
    @DependsOn("securityManager")
    @ConditionalOnMissingBean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultSecurityManager securityManager, Realm realm) {
        MyRealm myRealm = (MyRealm) realm;
        securityManager.setRealm(myRealm);
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);
        shiroFilter.setLoginUrl(properties.getLoginUrl());
        shiroFilter.setSuccessUrl(properties.getSuccessUrl());
        shiroFilter.setUnauthorizedUrl(properties.getUnauthorizedUrl());
        shiroFilter.setFilterChainDefinitionMap(properties.getFilterChainDefinitions());
        return shiroFilter;
    }
}
```
这里例子里说明，我们需要提供realm和shiroFilter两个bean配置给spring容器。
getShiroFilterFactoryBean方法返回ShiroFilterFactoryBean，将会把yml里面的配置读取到ShiroFilterFactoryBean中，然后把realm设置进securityManager。
yml配置有ShiroProperties类持有并提供给ShiroFilterFactoryBean。
当然，还需要配置几个其他配置，都在ShiroManager配置好了。

#### ShiroProperties：
```
import java.util.Map;
import org.springframework.boot.context.properties.ConfigurationProperties;
/**
 * Configuration properties for Shiro.
 */
@ConfigurationProperties(prefix = "shiro")
public class ShiroProperties {
    private Class<?> realm;
    private String loginUrl;
    private String successUrl;
    private String unauthorizedUrl;
    private Map<String, String> filterChainDefinitions;

}
```

#### ShiroManager：
```
import org.apache.shiro.cache.CacheManager;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.session.mgt.SessionManager;
import org.apache.shiro.session.mgt.eis.SessionDAO;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.DependsOn;

/**
 * Shiro Config Manager.
 */
public class ShiroManager {
    /**
     * 保证实现了Shiro内部lifecycle函数的bean执行
     */
    @Bean(name = "lifecycleBeanPostProcessor")
    @ConditionalOnMissingBean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
    @Bean(name = "defaultAdvisorAutoProxyCreator")
    @ConditionalOnMissingBean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;

    }
    /**
     * 用户授权信息Cache
     */
    @Bean(name = "cacheManager")
    @ConditionalOnMissingBean
    public CacheManager cacheManager() {
        return new MemoryConstrainedCacheManager();
    }
    @Bean(name = "securityManager")
    @ConditionalOnMissingBean
    public DefaultSecurityManager securityManager(CacheManager cacheManager) {
        DefaultSecurityManager sm = new DefaultWebSecurityManager();
        sm.setCacheManager(cacheManager);
        return sm;
    }
    @Bean
    @ConditionalOnMissingBean
    public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor(DefaultSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        aasa.setSecurityManager(securityManager);
        return new AuthorizationAttributeSourceAdvisor();
    }
}
```
CacheManager负责缓存，这里简单的使用默认的内存缓存管理MemoryConstrainedCacheManager。
如果需要ehcache，或者使用redis作为缓存，则只需要实现自己的CacheManager和SessionManager即可。

至此，springboot中引入shiro就完成了，其他更多具体使用方法，根据具体而定。

### 这里简单的提供一下在mvc的controller中做登录和登出怎么做。

LoginController：
```
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.IncorrectCredentialsException;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.subject.Subject;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController {
    @RequestMapping("/login")
    @ResponseBody
    public void login(HttpServletRequest req, HttpServletResponse resp, String username, String password) throws ServletException, IOException {
        Subject subject = SecurityUtils.getSubject();
        String error = null;
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        try {
            subject.login(token);
        } catch (UnknownAccountException e) {
            error = "用户名/密码错误";
        } catch (IncorrectCredentialsException e) {
            error = "用户名/密码错误";
        } catch (AuthenticationException e) {
            // 其他错误，比如锁定，如果想单独处理请单独catch处理
            error = "其他错误：" + e.getMessage();
        }
        if (error != null) {// 出错了，返回登录页面
            req.setAttribute("error", error);
            resp.sendRedirect("/forbidden.html");
        } else {// 登录成功
            resp.sendRedirect("/index.html");// 设置跳转的页面
        }
    }
    @RequestMapping(value = "/logout")
    @ResponseBody
    public void logout(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        Subject currentUser = SecurityUtils.getSubject();
        currentUser.logout();
        resp.sendRedirect("/index.html");
    }
}
```

done!