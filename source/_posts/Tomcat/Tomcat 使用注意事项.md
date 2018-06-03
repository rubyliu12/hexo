---
title: Tomcat 使用注意事项
categories:
  - Tomcat
tags: []
abbrlink: d13f7d34
date: 2018-05-08 09:35:05
---

# setenv.sh

在`Tomcat`的`bin`目录下添加`setenv.sh`文件，在启动`Tomcat`的时候，改脚本会被自动执行。一般在改脚本配置一些环境变量、`JVM`参数等

```sh
# @Author: YL
# @Date:   2017-12-18 12:15:52
# @Last Modified by:   YL
# @Last Modified time: 2017-12-18 12:16:03
# 本文件放到 Tomcat 的 bin 目录下

export LANG=en_US.UTF-8
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH

JAVA_OPT="$JAVA_OPT -Xss256k"
```



<!-- more -->



# Tomcat 安全配置相关

## server.xml

> port 改成 -1

```xml
<Server port="-1" shutdown="SHUTDOWN">
```



> autoDeploy 改成 false

```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="false">
```



> 注释掉 AJP Connector

```xml
<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
```



# server.xml Context 配置


```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="false">
		<Context path="/outserver" docBase="../../project/outserver.war" privileged="true" reloadable="false" />
      </Host>
```

这样配置，outserver.war会解压到tomcat/webapps下，也就是在webapps下生成outserver文件夹。

>  docBase应该指定对应的war包全路径，不应该指定路径，比如：../../project/outserver。
>
> 如果指定路径，有可以出现war包不会解压，到时项目没有更新的问题

