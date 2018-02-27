---
title: jar启动脚本
date: 2018-02-06 09:35:05
categories: [Linux]
tags: [Java, jar]
description:
permalink: linux-shell-jar-start
---

# jar启动脚本
```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-08-31 15:52:30
# @Last Modified by:   YL
# @Last Modified time: 2017-12-18 14:12:45

export LANG=en_US.UTF-8
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH

cd `dirname $0`
PROJECT_HOME=`pwd`
PROJECT_LIB=$PROJECT_HOME/lib
PROJECT_LOGS=$PROJECT_HOME/logs
STDOUT_FILE=$PROJECT_HOME/logs/stdout.log
PROJECT_JAR=$PROJECT_HOME/tisson-ftp.jar

# 执行检测的用户IP地址
WHO_AM_I=`who am i | awk '{print $5}' | sed 's/(//g' | sed 's/)//g'`

# date pattern
DATE_PATTERN="[$(date '+%Y-%m-%d %H:%M:%S')]-[$1]-[$WHO_AM_I] $PROJECT_JAR"

# pid
PROJECT_PID=`ps -ef | grep "$PROJECT_JAR" | grep -v grep | awk '{print $2}'`

function operate(){
  if [[ "$1" = "kill" ]]; then
    if [ -n "$PROJECT_PID" ]; then
      echo "$DATE_PATTERN kill $PROJECT_PID begining" | tee -a $STDOUT_FILE
      kill -9 $PROJECT_PID
      echo "$DATE_PATTERN kill $PROJECT_PID success" | tee -a $STDOUT_FILE
    else
      echo "$DATE_PATTERN is not alive" | tee -a $STDOUT_FILE
    fi
  elif [[ "$1" = "start" ]] ; then
    # cd $PROJECT_HOME
    # starting
    nohup java -server -Xms512m -Xmx512m -Xss256k -Dloader.path=$PROJECT_LIB -Djava.io.tmpdir=$PROJECT_LOGS -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=$PROJECT_LOGS/$PROJECT_PID.hprof -jar $PROJECT_JAR >> $STDOUT_FILE 2>&1 &
    PROJECT_PID=`ps -ef | grep "$PROJECT_JAR" | grep -v grep | awk '{print $2}'`
    echo "$DATE_PATTERN The $PROJECT_JAR started OK! pid: $PROJECT_PID" | tee -a $STDOUT_FILE
  else
    echo "$DATE_PATTERN is not support $1" | tee -a $STDOUT_FILE
  fi
}

if [[ "$1" = "start" || "$1" = "check" ]]; then
  if [ -n "$PROJECT_PID" ]; then
    echo "$DATE_PATTERN already started! pid: $PROJECT_PID" >> $STDOUT_FILE
    exit 1
  fi
  operate start
elif [[ "$1" = "" || "$1" = "restart" ]]; then
  operate kill
  echo "$DATE_PATTERN starting" | tee -a $STDOUT_FILE
  operate start
elif [[ "$1" = "kill" ]]; then
  operate kill
else
  echo "$DATE_PATTERN is not support $1" | tee -a $STDOUT_FILE
fi
```
