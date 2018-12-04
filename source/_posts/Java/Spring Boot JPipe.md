---
title: Java 实现 BigPipe
categories:
  - Java
tags:
  - jsp
  - freemarker
  - BigPipe
  - spring-boot
abbrlink: d7b08969
date: 2018-08-18 14:47:17
---
# JPipe Project
Spring Boot Project for JPipe(https://github.com/foreveryang321/jpipe)



## 开始
### 线程池配置
```yaml
jpipe:
  enabled: true
  pool:
    core-size: -1
    pre-start-all-core-threads: false
    max-size: 1024
    keep-alive: 6000
    queue-size: 1024
```

| 属性                       | 类型    | 是否必填 | 缺省值 | 说明              | 备注                                                         |
| -------------------------- | ------- | -------- | ------ | ----------------- | ------------------------------------------------------------ |
| core-size                  | int     | 否       | -1     | 核心线程数        | 最小空闲线程数，无论如何都会存活的最小线程数                 |
| max-size                   | int     | 否       | 1024   | 最大线程数        | JPipe 能创建用来处理 pagelet 的最大线程数                    |
| queue-size                 | int     | 否       | 1024   | 最大等待对列数    | 请求并发大于 max-size，则被放入队列等待                      |
| keep-alive                 | long    | 否       | 60000  | 最大空闲时间      | 超过这个空闲时间，且线程数大于 core-size 的，被回收直到线程数等于core-size |
| pre-start-all-core-threads | boolean | 否       | false  | 预热线程池        | 是否预先启动 core-size 个线程                                |
| enabled                    | boolean | 否       | true   | 是否开启JPipe配置 |                                                              |



###  SpringApplication 启动类

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import top.ylonline.boot.jpipe.autoconfigure.EnableJPipeConfiguration;

@SpringBootApplication
@EnableJPipeConfiguration
public class App extends SpringBootServletInitializer {

    /**
     * Common
     */
    private static SpringApplicationBuilder configureSpringBuilder(SpringApplicationBuilder builder) {
        // builder.listeners(new EnvironmentPreparedEventListener());
        return builder.sources(AppJsp.class);
    }

    /**
     * for WAR deploy
     */
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return configureSpringBuilder(builder);
    }

    /**
     * for JAR deploy
     */
    public static void main(String[] args) {
        configureSpringBuilder(new SpringApplicationBuilder()).run(args);
    }
}

```

