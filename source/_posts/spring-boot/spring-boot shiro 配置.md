---
title: spring-boot shiro 配置
categories:
  - Java
tags:
  - spring-boot
  - shiro
abbrlink: ae8ab207
date: 2018-11-20 15:34:07
---



# 初始化

使用@Configuration配置shiro无状态登录时出现的问题，在subject.login之后当前线程重新绑定了一个假定subject，isAuthenticated。

这里自定义的访问拦截器的创建需要放在shiroFilter之后，如下：

```java
/**
  * Shiro 的 Web 过滤器链
  */
@Bean("shiroFilter")
public ShiroFilterFactoryBean shiroFilter() {
    ShiroFilterFactoryBean filter = new ShiroFilterFactoryBean();
    filter.setSecurityManager(securityManager());
 
    Map<String, Filter> filters = new LinkedHashMap<String, Filter>();
    // 无状态授权器
    filters.put("statelessAuthc", statelessAuthcFilter());
    filter.setFilters(filters);
 
    /**
     * 配置shiro拦截器链
     */
    // filterChainDefinitionMap 必须是 LinkedHashMap 因为它必须保证有序
    Map<String, String> chain = new LinkedHashMap<String, String>();
    // anon-表示可以匿名访问， authc-表示需要认证才可以访问
    // 因为禁用了 Session，所以这里不能使用 authc 了，否则会报 DisabledSessionException 异常
    chain.put("/services/*", "statelessAuthc");
    chain.put("/**", "anon");
    filter.setFilterChainDefinitionMap(chain);
    return filter;
}

@Bean
public AccessControlFilter statelessAuthcFilter() {
    return new StatelessAccessControlFilter();
}
```

