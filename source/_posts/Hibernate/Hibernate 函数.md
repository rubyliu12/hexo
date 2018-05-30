---
title: Hibernate 函数
categories:
  - Hibernate
tags:
  - Hibernate
abbrlink: 35958b11
date: 2018-05-30 09:21:36
---



HQL(Hibernate Query Language) 提供了丰富的、灵活的查询特性，提供了类似标准SQL语句的查询方式，同时也提供了面向对象的封装。以下是HQL的一些常用函数，比如时间函数、字符串函数等。



<!-- more -->



# 时间函数

| 函数名称            | 说明                | 支持       | 使用方法            | 备注             |
| ------------------- | ------------------- | ---------- | ------------------- | ---------------- |
| CURRENT_DATE()      | 数据库当前日期      | JPAQL、HQL | CURRENT_DATE()      |                  |
| CURRENT_TIME()      | 数据库当前时间      | JPAQL、HQL | CURRENT_TIME()      |                  |
| CURRENT_TIMESTAMP() | 数据库当前日期+时间 | JPAQL、HQL | CURRENT_TIMESTAMP() |                  |
| SECOND(d)           | 日期中提取秒        | HQL        | SECOND(时间字段)    | 空的时候返回null |
| MINUTE(d)           | 日期中提取分        | HQL        | MINUTE(时间字段)    | 空的时候返回null |
| HOUR(d)             | 日期中提取小时      | HQL        | HOUR(时间字段)      | 空的时候返回null |
| DAY(d)              | 日期中提取天        | HQL        | DAY(时间字段)       | 空的时候返回null |
| MONTH(d)            | 日期中提取月        | HQL        | MONTH(时间字段)     | 空的时候返回null |
| YEAR(d)             | 日期中提取年        | HQL        | YEAR(时间字段)      | 空的时候返回null |



# 字符串函数

| 函数名称                   | 说明           | 支持       | 使用方法 | 备注 |
| -------------------------- | -------------- | ---------- | -------- | ---- |
| SUBSTRIGN(s,offset,length) | 字符截取       | JPAQL、HQL |          |      |
| TRIM(s)                    | 去掉两端的空格 | JPAQL、HQL |          |      |
| LOWER(s)                   | 转换小写       | JPAQL、HQL |          |      |
| UPPER(s)                   | 转换大写       | JPAQL、HQL |          |      |
| LENGTH(s)                  | 字符长度       | JPAQL、HQL |          |      |
| CONCAT(s1,s2)              | 连接字符串     | JPAQL、HQL |          |      |



# 数学函数

| 函数名称    | 说明     | 支持       | 使用方法                                         | 备注           |
| ----------- | -------- | ---------- | ------------------------------------------------ | -------------- |
| ABS(n)      | 取绝对值 | JPAQL、HQL | ABS(col_name[数字类型对象属性])                  |                |
| SQRT(n)     | 取平方根 | JPAQL、HQL | SQRT(col_name[数字类型对象属性])                 |                |
| MOD(n1, n2) | 取余数   | JPAQL、HQL | MOD([对象属性(数字)或值],[对象属性（数字）或值]) | 数字必须是整型 |



# 集合函数

| 函数名称      | 说明                 | 支持         | 使用方法 | 备注 |
| ------------- | -------------------- | ------------ | -------- | ---- |
| MAX(c)        | 最大值               | JPQHQL 、HQL |          |      |
| MIN(c)        | 最小值               | JPQHQL 、HQL |          |      |
| COUNT(c)      | 返回计数             | JPQHQL 、HQL |          |      |
| SIZE(c)       | 方法集合内对象数量   | JPAQL HQL    |          |      |
| MINELEMENT(c) | 返回集合中最小元素   | HQL          |          |      |
| MAXELEMENT(c) | 返回集合中最大元素   | HQL          |          |      |
| MININDEX(c)   | 返回索引集合最小索引 | HQL          |          |      |
| MAXINDEX(c)   | 返回索引集合最大索引 | HQL          |          |      |


