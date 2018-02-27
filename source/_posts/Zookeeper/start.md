---
title: Zookeeper 启动脚本
date: 2018-02-06 09:35:05
categories: [Zookeeper]
tags: []
description:
permalink: zookeeper-start-shell
---

# start
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-06-28 10:23:20
# @Last Modified by:   YL
# @Last Modified time: 2017-11-01 09:37:41
# 此文件放到 zookeeper 的 bin 目录下

cd `dirname $0`
NODE_BIN_DIR=`pwd`
NODE_LIB_DIR=$NODE_BIN_DIR/../lib/
cd ..
NODE_HOME=`pwd`

# date pattern
DATE_PATTERN="[$(date '+%Y-%m-%d %H:%M:%S')]"

# mkdir logs dir
NODE_DATA_LOG_DIR=`sed '/dataLogDir/!d;s/.*=//' conf/zoo.cfg | tr -d '\r'`
NODE_LOGS_DIR=""
if [ -n "$NODE_DATA_LOG_DIR" ]; then
  NODE_LOGS_DIR=$NODE_DATA_LOG_DIR
else
  NODE_LOGS_DIR=$NODE_HOME/logs
fi
if [ ! -d $NODE_LOGS_DIR ]; then
  mkdir $NODE_LOGS_DIR
fi
STDOUT_FILE=$NODE_LOGS_DIR/stdout.log

echo "$DATE_PATTERN NODE_BIN_DIR: $NODE_BIN_DIR" | tee -a $STDOUT_FILE

# pid
NODE_PID=`ps -ef | grep "$NODE_LIB_DIR" | grep -v grep | awk '{print $2}'`
if [ -n "$NODE_PID" ]; then
  echo "$DATE_PATTERN ERROR: The $NODE_LIB_DIR already started! pid: $NODE_PID" | tee -a $STDOUT_FILE
  exit 1
fi

# port
NODE_PORT=`sed '/clientPort/!d;s/.*=//' conf/zoo.cfg | tr -d '\r'`
if [ -n "$NODE_PORT" ]; then
  NODE_PORT_COUNT=`netstat -tln | grep $NODE_PORT | wc -l`
  if [ $NODE_PORT_COUNT -gt 0 ]; then
    echo "$DATE_PATTERN ERROR: The $NODE_LIB_DIR port $NODE_PORT already used!" | tee -a $STDOUT_FILE
    exit 1
  fi
fi

# starting
cd $NODE_BIN_DIR
./zkServer.sh start

NODE_PID=`ps -ef | grep "$NODE_LIB_DIR" | grep -v grep | awk '{print $2}'`
echo "$DATE_PATTERN The $NODE_LIB_DIR started OK! pid: $NODE_PID" | tee -a $STDOUT_FILE
```
