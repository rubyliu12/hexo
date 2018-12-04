---
title: spring cache 注解
categories:
  - Java
tags:
  - spring
  - cache
abbrlink: 3bef8a86
date: 2018-07-18 16:43:05
---





# Cacheable

|               |      |                                                              |
| ------------- | ---- | ------------------------------------------------------------ |
| value         |      | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个       |
| cacheNames    |      | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个       |
| key           |      | 缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合 |
| keyGenerator  |      |                                                              |
| cacheManager  |      |                                                              |
| cacheResolver |      |                                                              |
| condition     |      | 缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才进行缓存 |
| unless        |      | 缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 false才进行缓存 |
| sync          |      |                                                              |





# CachePut











# CachEvic



