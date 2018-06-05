---
title: druid 打印 sql 日志
categories:
  - Java
tags:
  - alibaba
  - druid
  - spring-boot
abbrlink: bee514fe
date: 2018-06-05 11:35:05
---



# druid 打印 sql 日志配置

> 依赖包

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```



<!-- more -->



> 在 application.yml 中配置 slf4j.enabled=true

```yaml
spring:
  datasource:
    url: jdbc:oracle:thin:@ip:port
    username: admin
    password: admin
    druid:
      initial-size: 3
      min-idle: 3
      max-active: 10
      max-wait: 60000
      keep-alive: true
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 'x' FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      filter:
        wall:
          enabled: true
        stat:
          enabled: false
        slf4j:
          enabled: true
          connection-log-enabled: false
```

> 在 logback.xml 中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <property name="logger.charset" value="UTF-8"/>
    <property name="logger.path" value="./logs"/>
    <property name="logger.pattern"
              value="%d{yy-MM-dd HH:mm:ss} %p %t %c{2}.%M\\(%L\\) | %m%n"/>
    <property name="logger.maxHistory" value="7"/>

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder charset="${logger.charset}">
            <pattern>${logger.pattern}</pattern>
        </encoder>
    </appender>

    <!-- druid -->
    <logger name="druid.sql.Connection" level="debug"/>
    <logger name="druid.sql.DataSource" level="debug"/>
    <logger name="druid.sql.Statement" level="debug"/>
    <logger name="druid.sql.ResultSet" level="debug"/>

    <root level="info">
        <appender-ref ref="stdout"/>
    </root>
</configuration>
```

> sql 日志

```tex
[18-06-05 11:56:40 DEBUG] main d.s.Statement.statementLog(134) | {conn-10003, pstmt-20002} created. select user0_.id as id1_0_, user0_.openid as openid2_0_ from user user0_ where user0_.openid=?
[18-06-05 11:56:40 DEBUG] main d.s.Statement.statementLog(134) | {conn-10003, pstmt-20002} Parameters : [oTlSg04-KUsQZc_OCKyDCALMizxw]
[18-06-05 11:56:40 DEBUG] main d.s.Statement.statementLog(134) | {conn-10003, pstmt-20002} Types : [VARCHAR]
[18-06-05 11:56:40 DEBUG] main d.s.Statement.statementLog(134) | {conn-10003, pstmt-20002, rs-50001} query executed. 20.335752 millis. select user0_.id as id1_0_, user0_.openid as openid2_0_ from user user0_ where user0_.openid=?
[18-06-05 11:56:40 DEBUG] main d.s.ResultSet.resultSetLog(139) | {conn-10003, pstmt-20002, rs-50001} open
[18-06-05 11:56:40 DEBUG] main d.s.ResultSet.resultSetLog(139) | {conn-10003, pstmt-20002, rs-50001} Header: [ID1_0_, OPENID2_0_]
[18-06-05 11:56:40 DEBUG] main d.s.ResultSet.resultSetLog(139) | {conn-10003, pstmt-20002, rs-50001} Result: [14248992, oTlSg04-KUsQZc_OCKyDCALMizxw]
[18-06-05 11:56:40 DEBUG] main d.s.ResultSet.resultSetLog(139) | {conn-10003, pstmt-20002, rs-50001} closed
[18-06-05 11:56:40 DEBUG] main d.s.Statement.statementLog(134) | {conn-10003, pstmt-20002} clearParameters. 
```

