---
title: Tomcat 启动脚本
date: 2018-02-06 09:35:05
categories: [Tomcat]
tags: [shell]
description:
permalink: tomcat-start-shell
---

# Tomcat 启动脚本

## start.sh

```bash
#!/usr/bin/env bash

# @Author: YL
# @Date:   2017-01-25 14:50:43
# @Last Modified by:   YL
# @Last Modified time: 2017-12-18 14:12:23

cd `dirname $0`
PROJECT_HOME=`pwd`

# Tomcat 服务器路径
TC_HOME="$PROJECT_HOME/tomcat"
# crontab 检测日记文件
TC_STDOUT_FILE="$TC_HOME/logs/stdout.out"
# 检测 Tomcat 是否启动（进程ID）
TC_PID=`ps -ef | grep $TC_HOME/bin | grep -v grep | awk '{print $2}'`

# 执行检测的用户IP地址
WHO_AM_I=`who am i | awk '{print $5}' | sed 's/(//g' | sed 's/)//g'`

# 检测输出的日记格式
DATE_PATTERN="[$(date '+%Y-%m-%d %H:%M:%S')]-[$1]-[$WHO_AM_I] $TC_HOME"

function operate(){
  if [[ "$1" = "kill" ]] ; then
    if [[ "$TC_PID" = "" ]] ; then
      echo "$DATE_PATTERN is not alive" | tee -a $TC_STDOUT_FILE
    else
      echo "$DATE_PATTERN kill $TC_PID begining" | tee -a $TC_STDOUT_FILE
      kill -9 $TC_PID
      echo "$DATE_PATTERN kill $TC_PID success" | tee -a $TC_STDOUT_FILE
    fi
  elif [[ "$1" = "start" ]]; then
    cd $TC_HOME/bin
    sh startup.sh
  else
    echo "$DATE_PATTERN is not support $1" | tee -a $TC_STDOUT_FILE
  fi
}
if [[ "$1" = "" || "$1" = "restart" || "$1" = "start" ]] ; then
  operate kill
  echo "$DATE_PATTERN starting" | tee -a $TC_STDOUT_FILE
  operate start
elif [[ "$1" = "kill" ]] ; then
  operate kill
elif [[ "$1" = "check" ]] ; then
  if [[ "$TC_PID" = "" ]] ; then
    echo "$DATE_PATTERN starting" >> $TC_STDOUT_FILE
    operate start
  else
    echo "$DATE_PATTERN pid $TC_PID" >> $TC_STDOUT_FILE
  fi
else
  echo "$DATE_PATTERN is not support $1" | tee -a $TC_STDOUT_FILE
fi
```



## 使用说明

```sh
sh start.sh start # 重启
sh start.sh # 重启
sh start.sh restart # 重启
sh start.sh check # 检测服务是否启动，没有启动，则启动之，否则不做其他操作
sh start.sh kill # kill掉已启动的服务进程
```