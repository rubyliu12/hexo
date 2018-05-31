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


总记录数
getTotalElements ---> 21

总页数
getTotalPages ---> 7

查询记录列表
getContent ---> [User(id=1, name=AAA, age=20, status=DEFAULT, createDate=null, addressId=2), User(id=2, name=AAA, age=20, status=DEFAULT, createDate=null, addressId=null), User(id=3, name=AAA, age=20, status=DEFAULT, createDate=null, addressId=null)]

当前页数，从0开始
getNumber ---> 0

当前页码中的记录数
getNumberOfElements ---> 3

每页记录数
getSize ---> 3

