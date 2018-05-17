---
title: IntelliJ IDEA 插件
categories:
  - Intellij
tags:
  - IDE
  - maven
abbrlink: b7ca9e0f
date: 2017-08-02 11:07:09
---

windows环境下，Intellij idea12中maven操作时，控制台中文乱码问题（编译报错或者clean install时出现的其他错误描述乱码）

在cmd中mvn中文正常显示,log4j打印日志也是ok的。

解决方法：

Setting -> maven -> runner
VMoptions: -Dfile.encoding=UTF-8

在idea的vm.propertis加入
-Dfile.encoding=UTF-8
