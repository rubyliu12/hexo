---
title: spring-boot cxf 集成
categories:
  - Java
tags:
  - cxf
  - spring-boot
abbrlink: b0385cf0
date: 2018-06-02 18:35:05
---



# 坑一

```java
@Bean
public AegisDatabinding aegisDatabinding() {
    return new AegisDatabinding();
}

/**
  * 这个要配置成多例模式
  */
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public org.apache.cxf.jaxws.support.JaxWsServiceFactoryBean jaxWsServiceFactoryBean() {
    JaxWsServiceFactoryBean sf = new JaxWsServiceFactoryBean();
    sf.setWrapped(true);
    sf.setDataBinding(aegisDatabinding());
    return sf;
}

@Bean
public Endpoint userEndpoint(UserService userService) {
    EndpointImpl endpoint = new EndpointImpl(userService);
    endpoint.setServiceFactory(jaxWsServiceFactoryBean());
    endpoint.publish("/imCheckRelatedWeixinService.ws");
    return endpoint;
}
```



这里 endpoint.publish 要在最后，否则获取到的 serviceFactory、dataBinding 等参数是空的