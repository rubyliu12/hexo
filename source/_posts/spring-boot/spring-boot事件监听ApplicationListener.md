---
title: spring-boot 事件监听 ApplicationListener
categories:
  - Java
tags:
  - spring-boot
abbrlink: ba3065f8
date: 2018-07-06 16:11:20
---



# 介绍

SpringBoot 为 ApplicationContextEvent 提供了四种事件：

 *  ApplicationStartedEvent ：spring boot 启动开始时执行的事件
 *  ApplicationEnvironmentPreparedEvent：spring boot 对应 Enviroment 已经准备完毕，但此时上下文 context 还没有创建
 *  ApplicationPreparedEvent：spring boot 上下文 context 创建完成，但此时 spring 中的 bean 是没有完全加载完成的
 *  ApplicationFailedEvent：spring boot启动异常时执行事件



<!-- more -->



# ApplicationEnvironmentPreparedEvent事件监听

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 * 环境配置事件监听器
 *
 * @author Created by YL on 2018/7/5
 */
@Slf4j
public class EnvironmentPreparedEventListener implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {

    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        ConfigurableEnvironment environment = event.getEnvironment();
        // 获取 spring.profiles.active 的值
        String activeProfile = environment.getActiveProfiles()[0];
        log.info("激活环境 ---> activeProfile: {}", activeProfile);
    }
}

```

> 这里要注意注册`ApplicationEnvironmentPreparedEvent`监听器的方式

如果是 jar 环境部署

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;

import java.util.Collections;

/**
 * @author Created by YL on 2018/7/5
 */
@SpringBootApplication
public class App {

    /**
     * Common
     */
    private static SpringApplicationBuilder configureSpringBuilder(SpringApplicationBuilder builder) {
        builder.listeners(new EnvironmentPreparedEventListener());
        return builder.sources(App.class);
    }

    /**
     * for JAR deploy
     */
    public static void main(String[] args) {
        configureSpringBuilder(new SpringApplicationBuilder())
            .run(args);
        synchronized (App.class) {
           while (true) {
                try {
                    App.class.wait();
                } catch (Exception e) {
                    e.printStackTrace();
                    break;
                }
            }
        }
    }
}
```

或者

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;

import java.util.Collections;

/**
 * @author Created by YL on 2018/7/5
 */
@SpringBootApplication
public class App {
    
    /**
     * for JAR deploy
     */
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.addListeners(new EnvironmentPreparedEventListener());
        app.run(args);
        synchronized (App.class) {
           while (true) {
                try {
                    App.class.wait();
                } catch (Exception e) {
                    e.printStackTrace();
                    break;
                }
            }
        }
    }
}
```

如果是 war 环境部署

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

import java.util.Collections;

/**
 * @author Created by YL on 2018/7/2
 */
@SpringBootApplication
public class App extends SpringBootServletInitializer {

    /**
     * Common
     */
    private static SpringApplicationBuilder configureSpringBuilder(SpringApplicationBuilder builder) {
        builder.listeners(new EnvironmentPreparedEventListener());
        return builder.sources(App.class);
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
        configureSpringBuilder(new SpringApplicationBuilder())
            .run(args);
        synchronized (App.class) {
           while (true) {
                try {
                    App.class.wait();
                } catch (Exception e) {
                    e.printStackTrace();
                    break;
                }
            }
        }
    }
}

```

