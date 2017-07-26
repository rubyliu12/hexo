---
title: hibernate格式化输出sql语句（spring-data-jpa）
date: 2017-07-26 15:47:36
categories: [hibernate]
tags: [spring-boot, hibernate, sql]
description:
permalink: hibernate-format-sql
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
