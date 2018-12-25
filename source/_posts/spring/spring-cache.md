---
title: spring cache 常用缓存注解介绍
categories:
  - Java
tags:
  - spring
  - cache
abbrlink: 3bef8a86
date: 2018-07-18 16:43:05
---





# spring-cache 常用缓存注解介绍

| 包地址                               | 注解名      | 作用域   | 作用                      |
| ------------------------------------ | ----------- | -------- | ------------------------- |
| org.springframework.cache.annotation | CacheConfig | 类级别   | s设置缓存的公共配置       |
|                                      | Cacheable   | 方法级别 | 缓存读取操作              |
|                                      | CacheEvict  | 方法级别 | 缓存失效操作              |
|                                      | CachePut    | 方法级别 | 缓存更新操作              |
|                                      | Caching     | 方法级别 | h混合读取、失效、更新操作 |



<!-- more -->



# CacheConfig

主要作用是全局配置，比如配置缓存名称（cacheNames），只需要在类上面使用这个注解配置一次，类下面的方法就默认使用全局配置了。



# Cacheable

这个注解是最重要的，主要实现的是功能在进行读操作的时候，先从缓存中查询，如果查询不到，再走业务流程获取数据，获取成功后，保存到缓存中。



| 属性          | 默认值 | 描述                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| value         |        | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个       |
| cacheNames    |        | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个       |
| key           |        | 缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合 |
| keyGenerator  |        |                                                              |
| cacheManager  |        |                                                              |
| cacheResolver |        |                                                              |
| condition     |        | 缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才进行缓存。不能使用返回结果 |
| unless        |        | 不缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 false才进行缓存。可以使用返回结果 |
| sync          |        |                                                              |



# CacheEvict

这个注解主要是配合`@Cacheable`一起使用，它的主要作用是清除缓存。当进行一些更新、删除操作时，就需要删除缓存。如果不删除缓存就会出现读取不到最新数据的情况。它有一个重要的数据`allEntries`默认是false。当true是，会清除所有缓存，而不以指定的key为主。



# CachePut

> 这个注解总是会把数据缓存，而不会每次检测缓存是否存在



# Caching

这个注解是`Cacheable`、`CachePut`、`CacheEvict`的综合体，可以使用这个注解包含以上3个注解。





# 



