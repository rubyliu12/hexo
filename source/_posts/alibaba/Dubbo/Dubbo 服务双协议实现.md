---
title: Dubbo 服务双协议实现
categories:
  - Java
tags:
  - alibaba
  - Dubbo
  - spring-boot
abbrlink: 90506ac1
date: 2018-09-18 10:35:05
---



# 在 application.yml 中添加如下配置

首先要开启多协议的配置开关，再通过 protocols 指定多协议

```yaml
dubbo:
  config:
  	# 开启多配置支持
    multiple: true
  # 多协议配置
  protocols:
    dubbo:
      name: dubbo
      port: 20885
      server: netty4
    rest:
      name: rest
      port: 8080
      server: netty
```

> - 开启了多协议之后，可以使用`@Service(protocol = {"dubbo", "rest"})`指定协议类型
> - 如果不需要多协议，也要明确指定其中一个协议`@Service(protocol = {"dubbo"})`
> - `rest.server`可以是`servlet`、`jetty`、`tomcat`、`netty`，这里以`netty4`为例
>
>
>
> 对比单协议的配置
>
> ```yaml
> dubbo:
>   protocol:
>     name: dubbo
>     port: 20885
> ```
>
> 如果是单协议配置，不能这样指定协议`@Service(protocol = {"dubbo"})`，否则会报错找不到`dubbo compent`



要在`pom.xml`引入 `resteasy` 的依赖

```xml
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-jaxrs</artifactId>
    <version>3.0.26.Final</version>
</dependency>
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-netty4</artifactId>
    <version>3.0.26.Final</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```



<!-- more --> 



# Dubbo Service 指定多个协议

```java
import com.alibaba.dubbo.config.annotation.Service;

@Service(
        protocol = {"dubbo", "rest"}
)
@javax.ws.rs.Path("/user")
public class UserServiceImpl implements UserService {

    @javax.ws.rs.GET
    @javax.ws.rs.Path("/say-hello")
    // @Produces({javax.ws.rs.core.MediaType.APPLICATION_JSON})
    public String sayHello(@javax.ws.rs.QueryParam("name") String name) {
        return "Hello, " + name + "!";
    }
}
```



# 验证 rest 服务

```shell
$ curl http://localhost:8080/demo/say-hello?name=forever
Hello, forever!
```

