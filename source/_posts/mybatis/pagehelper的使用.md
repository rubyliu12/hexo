---
title: pagehelper 使用笔记
categories:
  - mybatis
tags:
  - mybatis
  - pagehelper
abbrlink: 4f0d5500
date: 2017-08-01 09:21:36
---
主要是记录一些在使用`mybatis` + `pagehelper`过程中遇到的一些问题，和要注意的知识点
# PageHelper 分页
```java
// 获取第1页，10条内容，默认查询总数count
PageHelper.startPage(1, 10);

// 获取第1页，10条内容，不查询总数count
PageHelper.startPage(1, 10, false);
```
<!-- more -->
