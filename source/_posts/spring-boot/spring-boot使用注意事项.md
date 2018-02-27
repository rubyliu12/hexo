---
title: spring boot issues
date: 2017-09-11 14:11:20
categories: [Java]
tags: [spring-boot]
description:
permalink: spring-boot-issues
---
### 兼容问题

spring-boot-starter-parent切换回1.5.2.RELEASE版本

因为1.5.4.RELEASE、1.5.5.RELEASE、1.5.6.RELEASE、1.5.7.RELEASE、2.0.0.M1、2.0.0.M2、2.0.0.M3都存在使用spring-boot-starter-data-redis时，会出现异常

`java.lang.NoSuchMethodError: org.springframework.data.repository.config.AnnotationRepositoryConfigurationSource.<init>`异常

请查看[https://github.com/spring-projects/spring-boot/issues/9606](https://github.com/spring-projects/spring-boot/issues/9606)
