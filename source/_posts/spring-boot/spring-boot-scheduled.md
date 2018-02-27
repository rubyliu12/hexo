---
title: spring boot 定时器
date: 2017-03-21 15:37:27
categories: [Java]
tags: [spring-boot, task, schedule]
description:
permalink: spring-boot-schedule
---

在pom.xml中加入
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>1.5.2.RELEASE</version>
</dependency>
```
<!-- more -->
定时器类
```java
@Component
public class ArticleSummaryScheduled {

    //  等同于fixedRate
    @Scheduled(cron = "0/15 * * * * ?")
    // 上一次开始执行时间点之后6s再次执行
    //    @Scheduled(fixedRate = 6000)
    // 上一次执行完毕时间点之后6s再次执行
    //    @Scheduled(fixedDelay = 6000)
    public void process() {
        System.out.println(new Date());
    }
}
```

在main方法类上面使用 `@EnableScheduling` 注解
```java
@SpringBootApplication
// 定时器启动类
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

只要上面的配置，就可以启动一个定时器了
