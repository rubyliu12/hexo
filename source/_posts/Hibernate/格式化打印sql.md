---
title: hibernate格式化输出sql语句（spring-data-jpa）
categories:
  - Hibernate
tags:
  - spring-boot
  - Hibernate
  - sql
abbrlink: 723691b5
date: 2017-07-26 15:47:36
---
以下配置是基于 spring-boot 的 application.yml 配置
```yml
spring:
  jpa:
#    hibernate:
#      ddl-auto: create
    show-sql: true
#    database: mysql
    properties:
      hibernate:
        format_sql: true
```
