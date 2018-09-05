---
title: Spring Data JPA 使用
categories:
  - Java
tags:
  - spring-boot
  - jpa
abbrlink: 6cc2492a
date: 2017-09-11 14:11:20
---

# Page<T>

| 函数                | 描述               | 备注 |
| ------------------- | ------------------ | ---- |
| getTotalElements    | 总记录数           |      |
| getTotalPages       | 总页数             |      |
| getContent          | 查询记录列表       |      |
| getNumber           | 当前页数，从0开始  |      |
| getNumberOfElements | 当前页码中的记录数 |      |
| getSize             | 每页记录数         |      |



# Modifying注解

（1）可以通过自定义的 JPQL 完成 UPDATE 和 DELETE 操作。 注意： JPQL 不支持使用 INSERT

（2）在 @Query 注解中编写 JPQL 语句， 但必须使用 @Modifying 进行修饰. 以通知 Spring Data， 这是一个 UPDATE 或 DELETE 操作

（3）UPDATE 或 DELETE 操作需要使用事务，此时需要定义 Service 层，在 Service 层的方法上添加事务操作

（4）默认情况下， Spring Data 的每个方法上有事务， 但都是一个只读事务。 他们不能完成修改操作