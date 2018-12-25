---
title: Dubbo 服务双协议实现
categories:
  - Java
tags:
  - alibaba
  - Dubbo
  - spring-boot
abbrlink: 90506ac1
date: 2018-12-18 10:35:05
---



# 在 application.properties中添加如下配置

```properties
# Enable multiple config bindings
dubbo.config.multiple =true

# Protocol Config
dubbo.protocols.dubbo.name=dubbo
dubbo.protocols.dubbo.port=20885
dubbo.protocols.dubbo.status=server

dubbo.protocols.rest.name=rest
dubbo.protocols.rest.port=8080
```



<!-- more --> 



# 在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-jaxrs</artifactId>
    <version>3.0.19.Final</version>
</dependency>

<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>

<dependency>
    <groupId>org.mortbay.jetty</groupId>
    <artifactId>jetty</artifactId>
    <version>6.1.26</version>
    <exclusions>
        <exclusion>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>servlet-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



# Dubbo Service 指定多个协议

```java
@Service(
        version = "${user.service.version}",
        protocol = {"dubbo", "rest"},
        registry = "${dubbo.registry.id}"
)
@Path("user")
public class UserServiceImpl implements UserService {

    @GET
    @Path("/say-hello")
    public String sayHello(@QueryParam("name") String name) {
        return "Hello, " + name + "!";
    }
}
```



# 验证 rest 服务

```shell
$ curl http://localhost:8888/demo/say-hello?name=forever
Hello, forever!
```

