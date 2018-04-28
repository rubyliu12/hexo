---
title: zookeeper evn 环境变量配置脚本
date: 2018-02-06 09:35:05
categories: [zookeeper]
tags: []
description:
permalink: zookeeper-env-shell
---

# zookeeper-env
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-10-31 10:27:16
# @Last Modified by:   YL
# @Last Modified time: 2017-11-01 09:36:43
# 此文件放到 zookeeper 的 conf 目录下

# java env
export JAVA_HOME=/usr/java/jdk1.8.0_131
export PATH=$JAVA_HOME/bin:$PATH
export JVMFLAGS="-Xms1536m -Xmx1536m $JVMFLAGS"

cd `dirname $0`
cd ..
NODE_HOME=`pwd`

# zookeeper env
ZOO_LOG_DIR=$NODE_HOME/logs
ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
```
