---
title: zookeeper 清理日志文件
date: 2018-02-06 09:35:05
categories: [zookeeper]
tags: []
description:
permalink: zookeeper-clear-shell
---

# clear
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-11-01 08:59:34
# @Last Modified by:   YL
# @Last Modified time: 2017-11-01 09:36:09
# 此文件放到 zookeeper 的 bin 目录下

cd `dirname $0`
cd ..
NODE_HOME=`pwd`

# snapshot
NODE_DATA_DIR=$NODE_HOME/data/version-2
# snapshot log
NODE_LOG_DIR=$NODE_HOME/logs/version-2
# zk log
NODE_LOGS=$NODE_HOME/logs

# 定义了删除对应目录中的文件，保留最新的 count 个文件，可以将他写到 crontab 中
count=3
count=$[$count+1]
ls -t $NODE_LOG_DIR/log.* | tail -n +$count | xargs rm -f
ls -t $NODE_DATA_DIR/snapshot.* | tail -n +$count | xargs rm -f
ls -t $NODE_LOGS/zookeeper.log.* | tail -n +$count | xargs rm -f

# date pattern
DATE_PATTERN="[$(date '+%Y-%m-%d %H:%M:%S')]"
STDOUT_FILE=$NODE_LOGS/stdout.log
echo "$DATE_PATTERN clear logs success" | tee -a $STDOUT_FILE
```
