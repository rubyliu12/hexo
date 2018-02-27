---
title: Tomcat setenv.sh
date: 2018-02-06 09:35:05
categories: [Tomcat]
tags: [shell]
description:
permalink: tomcat-setenv-shell
---

# Tomcat setenv脚本
```bash
# @Author: YL
# @Date:   2017-12-18 12:15:52
# @Last Modified by:   YL
# @Last Modified time: 2017-12-18 12:16:03
# 本文件放到 Tomcat 的 bin 目录下

export LANG=en_US.UTF-8
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH

JAVA_OPT=""
```
