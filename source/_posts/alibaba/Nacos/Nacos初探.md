---
title: Nacos 初探
categories:
  - Java
tags:
  - Nacos
  - spring-boot
  - alibaba
abbrlink: c706fd60
date: 2018-12-16 11:11:20
---





[Nacos](https://nacos.io/zh-cn/) 是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

> 以下内容都是基于spring-boot + nacos



# 依赖管理

项目：[nacos-spring-boot-project](https://github.com/nacos-group/nacos-spring-boot-project)

```xml
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.1</version>
</dependency>
```



<!-- more -->



# application.yml

```yaml
nacos:
  config:
    server-addr: 192.168.56.101:8848
#    namespace: '0278390f-21e8-40ae-b46d-fac0ebb7af97'
```

> namespace是命名空间 ID，不能配置命名空间名称。如果不配置namespace，默认使用public



# 在 Nacos 控制台新建配置

**Data ID**：szim.properties

**Group**：DEFAULT_GROUP

**配置内容**：

```properties
id=1
name=forever杨
```



# App.java

```java
import com.alibaba.nacos.api.config.annotation.NacosConfigListener;
import com.alibaba.nacos.api.config.annotation.NacosValue;
import com.alibaba.nacos.spring.context.annotation.config.EnableNacosConfig;
import com.alibaba.nacos.spring.context.annotation.config.NacosPropertySource;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import lombok.extern.slf4j.Slf4j;

@SpringBootApplication
@EnableNacosConfig
@NacosPropertySource(dataId = App.DATA_ID)
@Slf4j
public class App {
    static final String DATA_ID = "szim.properties";

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @NacosConfigListener(
            dataId = App.DATA_ID,
            timeout = 500
    )
    public void onChange(String newContent) throws Exception {
        log.info("onChange: {}", newContent);
    }

    @Bean
    @Order(Ordered.LOWEST_PRECEDENCE - 1)
    public CommandLineRunner secondCommandLineRunner() {
        return new SecondCommandLineRunner();
    }

    public static class SecondCommandLineRunner implements CommandLineRunner {
        @NacosValue("${id:unknown}")
        private String id;

        @NacosValue("${name:unknown}")
        private String name;

        @Override
        public void run(String... args) throws Exception {
            log.info("id: {}, name: {}", id, name);
        }
    }
}

```



# 支持的配置方式

## yaml文件

> 支持的格式

```yaml
spring.redis.host: 192.168.199.101
spring.redis.port: 6379
```



> 不支持的格式

```yaml
spring:
  redis:
    host: 192.168.199.101
    port: 6379
```

> 据说后续版本会支持这种格式



## properties文件

```properties
spring.redis.host = 192.168.199.101
spring.redis.port = 6379
```



## 读取配置

```java
@Value("${spring.redis.port}")
private String host;

@NacosValue(value = "${spring.redis.port}", autoRefreshed = true)
private String port;
```



# Spring @Value注解和 Nacos @NacosValue 注解

| 注解        | 是否支持动态更新 | 补充                         |
| ----------- | ---------------- | ---------------------------- |
| @Value      | 否               |                              |
| @NacosValue | 是               | 前提需配置autoRefreshed=true |