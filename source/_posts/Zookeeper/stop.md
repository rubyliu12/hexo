---
title: zookeeper stop脚本
date: 2018-02-06 09:35:05
categories: [zookeeper]
tags: []
description:
permalink: zookeeper-stop-shell
---

# stop
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-06-29 09:00:00
# @Last Modified by:   YL
# @Last Modified time: 2017-11-01 09:37:49
# 此文件放到 zookeeper 的 bin 目录下

cd `dirname $0`
./zkServer.sh stop
```
