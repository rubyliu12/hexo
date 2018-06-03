---
title: spring-data-jpa 使用笔记
categories:
  - Java
tags:
  - spring-data-jpa
  - Hibernate
abbrlink: 43f916ea
date: 2017-08-01 09:21:36
---
主要是记录一些在使用`spring-data-jpa` + `Hibernate`过程中遇到的一些问题，和要注意的知识点

# Pageable 和 PageRequest 分页
在Mysql、Oracle中分页从0开始
```java
Pageable pageable = new PageRequest(0, 10);
```


<!-- more -->



查看`org.springframework.data.domain.PageRequest`源码可知，分页从0开始

```java
public Pageable first() {
    return new PageRequest(0, this.getPageSize(), this.getSort());
}
```
