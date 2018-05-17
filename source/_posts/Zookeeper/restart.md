---
title: zookeeper重启脚本
categories:
  - zookeeper
tags: []
abbrlink: 6c1ad93d
date: 2018-02-06 09:35:05
---

# restart
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-06-29 09:00:10
# @Last Modified by:   YL
# @Last Modified time: 2017-11-01 09:37:30
# 此文件放到 zookeeper 的 bin 目录下

cd `dirname $0`

./zkServer.sh stop
sleep 3
./zkServer.sh start
```
