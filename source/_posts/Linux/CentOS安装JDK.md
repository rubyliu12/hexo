---
title: CentOS安装JDK
date: 2017-03-21 15:25:05
categories: [Linux]
tags: [Java, JDK]
description:
permalink: centos-jdk
---

## 1、准备
创建目录
```sh
$ mkdir /usr/java
$ cd /usr/java
```
下载 `jdk-8u73-linux-x64.tar.gz`，解压
```sh
$ tar -zxvf jdk-8u73-linux-x64.tar.gz
```
会解压到 `/usr/java/jdk1.8.0_73` 下
<!-- more -->
## 2、配置环境变量(两种方式二选一配置即可)
### 2.1 编辑 `/etc/profile` 文件

修改 `/etc/profile` 文件当本机仅仅作为开发使用时推荐使用这种方法，因为此种配置时所有用户的 `shell` 都有权使用这些环境变量，可能会给系统带来安全性问题。
```sh
$ vi /etc/profile
```
增加如下内容
```sh
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH
```
保存退出 `:wq`
```sh
$ . /etc/profile
$ echo $JAVA_HOME
```

### 2.2 编辑用户目录下的 `.bash_profile`
修改 `.bash_profile` 文件这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别，如果需要给某个用户权限使用这些环境变量，只需要修改其个人用户主目录下的 `.bash_profile` 文件就可以了
```sh
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH
```
然后执行使之生效
```sh
source .bash_profile
```

可以看到 `JAVA_HOME` 不是指向真实的 `jdk` 目录 `/usr/java/jdk1.8.0_73`，而是指向 `/usr/java/jdk`
# 3、创建软链接
为jdk真实目录创建一个命名更加间洁快捷方式
```sh
$ ln -s /usr/java/jdk1.8.0_73 /usr/java/jdk
$ source /etc/profile
#$ source .bash_profile
```
至此，配置完成
