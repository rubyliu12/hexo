---
title: JVM优化
date: 2017-05-23 21:47:17
categories: [Java]
tags: [JVM]
description:
permalink: jvm
---
阿里巴巴dubbo框架推荐配置参数，参数值根据情况调整
```shell
-server 
-Xmx2g 
-Xms2g 
-Xmn256m 
-XX:PermSize=128m 
-Xss256k 
-XX:+DisableExplicitGC 
-XX:+UseConcMarkSweepGC 
-XX:+CMSParallelRemarkEnabled 
-XX:+UseCMSCompactAtFullCollection 
-XX:LargePageSizeInBytes=128m 
-XX:+UseFastAccessorMethods 
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:CMSInitiatingOccupancyFraction=70
```
<!-- more -->
```sh
-server # 服务器模式
-Xms # JVM初始分配的堆内存，一般和Xmx配置成一样，以避免每次gc后JVM重新分配内存
-Xmx # JVM最大允许分配的堆内存
-Xmn # 年轻代内存大小，整个JVM内存=年轻代+年老代+持久代
-XX:PermSize=128m # 持久代内存大小
-Xss256k # 设置每个线程的栈内存大小
-XX:+DisableExplicitGC # 忽略手动调用GC，System.gc()的调用就会变成一个空调用，完全不触发GC
-XX:+UseConcMarkSweepGC # 并发标记清除（CMS）收集器
-XX:+CMSParallelRemarkEnabled # 降低标记停顿
-XX:+UseCMSCompactAtFullCollection # 在FULLGC的时候对年老代的压缩（CMS是不会移动内存的，因此，这个非常容易产生碎片，导致内存不够用，因此，内存的压缩这个时候就会被启用。增加这个参数是个好习惯。可能会影响性能，但是可以消除碎片）
-XX:LargePageSizeInBytes=128m # 内存页的大小（内存页的大小不可设置过大，会影响Perm的大小）
-XX:+UseFastAccessorMethods # 原始类型的快速优化
-XX:+UseCMSInitiatingOccupancyOnly # 使用手动定义初始化定义开始CMS收集（禁止hostspot自行触发CMS GC）
-XX:CMSInitiatingOccupancyFraction=70 # 使用CMS作为垃圾回收，使用70%后开始CMS收集（符合公式：CMSInitiatingOccupancyFraction <=((Xmx-Xmn)-(Xmn-Xmn/(SurvivorRatior+2)))/(Xmx-Xmn)*100，否则会报promontion faild错误）
```
