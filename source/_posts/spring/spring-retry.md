---
title: spring retry 使用
categories:
  - Java
tags:
  - spring
  - retry
abbrlink: 9cd0a544
date: 2018-07-18 16:43:05
---



**@Retryable**

被注解的方法发生异常时会重试 

| 参数       | 默认值 | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| value      | 空     | 指定发生的异常，进行重试                                     |
| include    | 空     | 指定异常重试。和value一样，默认空，当exclude也为空时，所有异常都重试 |
| exclude    | 空     | 指定异常不重试。默认空，当include也为空时，所有异常都重试    |
| maxAttemps | 3      | 重试次数                                                     |
| backoff    | 空     | 重试补偿机制                                                 |



<!-- more -->



**@Backoff**

| 参数       | 默认值 | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| delay      | 1000L  | 指定延迟时间                                                 |
| multiplier | 0.0D   | 指定延迟的倍数。比如delay=5000l,multiplier=2时，第一次重试为5秒后，第二次为10秒，第三次为20秒 |



**@Recover** 

当重试到达指定次数时，被注解的方法将被回调，可以在该方法中进行日志处理。需要注意的是发生的异常和入参类型一致时才会回调。



**示例**

```java
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

/**
 * @author Created by YL on 2018/7/18
 */
@Service
@Slf4j
public class RetryService {
    @Retryable(
            value = {Exception.class},
            maxAttempts = 10,
            backoff = @Backoff(delay = 10000L)
    )
    public String retry() throws Exception {
        throw new Exception("测试 spring retry");
    }

    @Recover
    public void exception(Exception e) {
        log.error("errmsg: {}", e.getMessage(), e);
    }
}


import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry
public class App {
    /**
     * Common
     */
    private static SpringApplicationBuilder configureSpringBuilder(SpringApplicationBuilder builder) {
        //builder.application().addListeners(new EnvironmentPreparedEventListener());
        builder.application().addPrimarySources(Collections.singletonList(App.class));
        return builder.sources(App.class);
    }

    // /**
    //  * for WAR deploy
    //  */
    // @Override
    // protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
    //     // // 外部容器部署 web 环境要在这里配置 EnvironmentPreparedEvent 监听器才生效
    //     // builder.application().addListeners(new EnvironmentPreparedEventListener());
    //     // return builder.sources(Application.class);
    //     return super.configure(configureSpringBuilder(builder));
    // }

    /**
     * for JAR deploy
     */
    public static void main(String[] args) {
        // SpringApplication.run(Application.class, args);
        SpringApplicationBuilder builder = configureSpringBuilder(new SpringApplicationBuilder());
        builder.application().run(args);
    }
}

```

