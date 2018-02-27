---
title: SQL中inner join、outer join和cross join的区别
date: 2017-08-02 08:43:05
categories: [sql]
tags: [mysql, oracle]
description:
permalink: sql-join
---
# SQL中inner join、outer join和cross join的区别


## INNER JOIN 产生的结果是AB的交集
SELECT * FROM TableA INNER JOIN TableB ON TableA.name = TableB.name

## LEFT [OUTER] JOIN 产生表A的完全集，而B表中匹配的则有值，没有匹配的则以null值取代
SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name

## RIGHT [OUTER] JOIN 产生表B的完全集，而A表中匹配的则有值，没有匹配的则以null值取代
SELECT * FROM TableA RIGHT OUTER JOIN TableB ON TableA.name = TableB.name

## FULL [OUTER] JOIN 产生A和B的并集。对于没有匹配的记录，则会以null做为值
SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name 
你可以通过is NULL将没有匹配的值找出来：
SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name
WHERE TableA.id IS null OR TableB.id IS null

## CROSS JOIN 把表A和表B的数据进行一个N*M的组合，即笛卡尔积。如本例会产生4*4=16条记录，在开发过程中我们肯定是要过滤数据，所以这种很少用
SELECT * FROM TableA CROSS JOIN TableB
