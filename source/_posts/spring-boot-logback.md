---
title: spring boot logback多环境配置
date: 2017-03-21 15:39:36
categories: [Java]
tags: [spring-boot, logback]
description:
permalink: spring-boot-logback
---

`application.yml` 配置 `profiles`
```properties
spring:
  profiles:
    active: dev
```

在使用 `logback-spring.xml` 中使用 `springProfile` 配置，使用 `logback-spring.xml` 可以获得更多spring boot个性化配置
<!-- more -->
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- For assistance related to logback-translator or configuration -->
<!-- files in general, please contact the logback user mailing list -->
<!-- at http://www.qos.ch/mailman/listinfo/logback-user -->
<!-- -->
<!-- For professional support please see -->
<!-- http://www.qos.ch/shop/products/professionalSupport -->
<!-- -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <property name="logger.charset" value="UTF-8"/>
    <property name="logger.path" value="${catalina.home}/logs"/>
    <property name="logger.pattern" value="[%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5p)] %yellow(%t) %cyan(%c.%M\\(%L\\)) | %m%n"/>
    <property name="logger.maxHistory" value="15"/>

    <!--&lt;!&ndash; https://logback.qos.ch/manual/layouts.html#coloring &ndash;&gt;
    &lt;!&ndash; 彩色日志依赖的渲染类 &ndash;&gt;
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    &lt;!&ndash; 彩色日志格式 &ndash;&gt;
    <property name="logger.pattern" value="${CONSOLE_LOG_PATTERN:-%clr([%d{yyyy-MM-dd HH:mm:ss} %clr(${LOG_LEVEL_PATTERN:-%5p})]){faint} %clr(%c.%M\\(%L\\)){cyan} | %m%n}" />-->

    <springProfile name="dev">
        <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
            <encoder charset="${logger.charset}">
                <pattern>${logger.pattern}</pattern>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="stdout"/>
        </root>
    </springProfile>
    <springProfile name="prod">
        <appender name="stdout"
                  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <File>${logger.path}/info.log</File>
            <encoder charset="${logger.charset}">
                <pattern>${logger.pattern}</pattern>
            </encoder>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>
                    ${logger.path}/info.log.%d{yyyy.MM.dd}
                </fileNamePattern>
                <maxHistory>${logger.maxHistory}</maxHistory>
            </rollingPolicy>
        </appender>
        <root level="info">
            <appender-ref ref="stdout"/>
        </root>
    </springProfile>
</configuration>
```
