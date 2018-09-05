---
title: Spring ImportBeanDefinitionRegistrar 使用
categories:
  - Java
tags:
  - spring
abbrlink: 8246f0e2
date: 2018-08-28 18:43:05
---

## EnableJPipeConfiguration

> 自定义注解

```java
import org.springframework.context.annotation.Import;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * enable bigPipe
 *
 * @author Created by YL on 2018/8/15
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Import({JPipeConfigurationRegistrar.class})
public @interface EnableJPipeConfiguration {
    /**
     * the maximum number of threads to allow in the pool
     */
    int maxPoolSize() default 1024;

    /**
     * when the number of threads is greater than the core, this is the maximum time that excess idle threads will
     * wait for new tasks before terminating.
     * 单位：s
     */
    int keepAliveTime() default 60;

    /**
     * the queue to use for holding tasks before they are executed.  This queue will hold only the {@code Runnable}
     * tasks submitted by the {@code execute}
     * method.
     */
    int workQueueSize() default 512;
}
```

## JPipeConfigurationRegistrar

> 读取自定义注解配置，注册 bean

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.annotation.AnnotationAttributes;
import org.springframework.core.type.AnnotationMetadata;

/**
 * @author Created by YL on 2018/8/16
 */
@Slf4j
public class JPipeConfigurationRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AnnotationAttributes attr = AnnotationAttributes.fromMap(metadata
                .getAnnotationAttributes(EnableJPipeConfiguration.class.getName()));
        if (attr != null) {
            int maxPoolSize = attr.getNumber("maxPoolSize");
            int keepAliveTime = attr.getNumber("keepAliveTime");
            int workQueueSize = attr.getNumber("workQueueSize");

            log.info("maxPoolSize: {}", maxPoolSize);
            log.info("keepAliveTime: {}", keepAliveTime);
            log.info("workQueueSize: {}", workQueueSize);
        }
    }
}

```