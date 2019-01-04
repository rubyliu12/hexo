---
title: Dubbo 多配置绑定
categories:
  - Java
tags:
  - alibaba
  - Dubbo
  - spring-boot
abbrlink: 90506ac1
date: 2018-09-18 10:35:05
---





本文主要阐述，[Dubbo](http://dubbo.apache.org/zh-cn/) 多协议、单协议多端口实现



<!-- more --> 



# 多协议实现

## 在 application.yml 中添加如下配置

首先要开启多协议的配置开关，再通过 protocols 指定多协议

```yaml
dubbo:
  config:
  	# 开启多个配置绑定
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



## Dubbo Service 指定多个协议

```java
import com.alibaba.dubbo.config.annotation.Service;

@Service(
        protocol = {"dubbo", "rest"}
)
// @Service
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

> @Service 不配置 protocol，默认使用所有协议





## 验证 rest 服务

```shell
$ curl http://localhost:8080/demo/say-hello?name=forever
Hello, forever!
```





# 单协议多端口实现

## 在 application.yml 中添加如下配置

首先要开启多协议的配置开关，再通过 protocols 指定多协议

```yaml
dubbo:
  config:
  	# 开启多个配置绑定
    multiple: true
  # 多协议配置
  protocols:
    dubbo:
      name: dubbo
      port: 20885
      server: netty4
    dubbo2:
      name: dubbo
      port: 20886
      server: netty4
```



## Dubbo Service 指定多个协议

```java
import com.alibaba.dubbo.config.annotation.Service;

// @Service(
//         protocol = {"dubbo", "dubbo2"}
// )
@Service
public class UserServiceImpl implements UserService {

    public String sayHello(String name) {
        return "Hello, " + name + "!";
    }
}
```

> @Service 不配置 protocol，默认使用所有协议