---
title: hibernate显示sql中的参数值（logback）
date: 2017-07-26 15:47:36
categories: [hibernate]
tags: [hibernate, sql]
description:
permalink: hibernate-show-parameters
---
以下是 logback 的日志输出配置，只要是在 logback 的配置文件中添加以下配置
```xml
<!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
 <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
```
<!-- more -->
完整的 logback 配置如下
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
    <property name="logger.pattern"
              value="[%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5p)] %yellow(%t) %cyan(%c.%M\\(%L\\)) | %m%n"/>
    <property name="logger.maxHistory" value="15"/>

    <!--&lt;!&ndash; https://logback.qos.ch/manual/layouts.html#coloring &ndash;&gt;
    &lt;!&ndash; 彩色日志依赖的渲染类 &ndash;&gt;
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    &lt;!&ndash; 彩色日志格式 &ndash;&gt;
    <property name="logger.pattern" value="${CONSOLE_LOG_PATTERN:-%clr([%d{yyyy-MM-dd HH:mm:ss} %clr(${LOG_LEVEL_PATTERN:-%5p})]){faint} %clr(%c.%M\\(%L\\)){cyan} | %m%n}" />-->

    <springProfile name="dev, local">
        <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
            <encoder charset="${logger.charset}">
                <pattern>${logger.pattern}</pattern>
            </encoder>
        </appender>

        <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
        <!--<logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG"/>
        <logger name="org.hibernate.SQL" level="DEBUG"/>
        <logger name="org.hibernate.type" level="INFO"/>
        <logger name="org.hibernate.engine.QueryParameters" level="DEBUG"/>
        <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG"/>-->
        <!--<logger name="org.apache.ibatis" level="debug"/>
        <logger name="java.sql.Connection" level="debug"/>
        <logger name="java.sql.Statement" level="debug"/>
        <logger name="java.sql.PreparedStatement" level="debug"/>-->
        <root level="info">
            <appender-ref ref="stdout"/>
        </root>
    </springProfile>
    <springProfile name="prod">
        <!--See http://logback.qos.ch/manual/appenders.html#RollingFileAppender -->
        <!--and http://logback.qos.ch/manual/appenders.html#TimeBasedRollingPolicy -->
        <!--for further documentation -->
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
        <appender name="warn"
                  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <encoder charset="${logger.charset}">
                <pattern>${logger.pattern}</pattern>
            </encoder>
            <File>${logger.path}/warn.log</File>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>WARN</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>
                    ${logger.path}/warn.log.%d{yyyy.MM.dd}
                </fileNamePattern>
                <maxHistory>${logger.maxHistory}</maxHistory>
            </rollingPolicy>
        </appender>
        <appender name="error"
                  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <encoder charset="${logger.charset}">
                <pattern>${logger.pattern}</pattern>
            </encoder>
            <File>${logger.path}/error.log</File>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>ERROR</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>
                    ${logger.path}/error.log.%d{yyyy.MM.dd}
                </fileNamePattern>
                <maxHistory>${logger.maxHistory}</maxHistory>
            </rollingPolicy>
        </appender>
        <logger name="warn" level="WARN"/>
        <logger name="error" level="ERROR"/>
        <root level="info">
            <appender-ref ref="warn"/>
            <appender-ref ref="error"/>
            <appender-ref ref="stdout"/>
        </root>
    </springProfile>
</configuration>
```
