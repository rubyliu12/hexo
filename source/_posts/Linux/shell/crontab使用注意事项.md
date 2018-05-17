---
title: Linux crontab 命令
categories:
  - Linux
tags:
  - shell
  - crontab
abbrlink: b0b79272
date: 2018-02-06 09:35:05
---

# crontab 使用注意事项
如果通过 crontab 执行
1、crontab 无法获取 jdk 变量，要在 java 命令之前写入jdk绝对路径
2、要把路径切换到要执行的 sh 路径下
注意：二者缺一不可

## 查找 jdk 路径的方法
- 1
```bash
$ echo $JAVA_HOME
/usr/java/jdk1.6.0_45/bin/java
```
- 2
```bash
$ which java
/usr/bin/java
$ ls -lrt /usr/bin/java #/usr/bin/java是which java查出来的路径
lrwxrwxrwx. 1 root root 22 Sep 16  2015 /usr/bin/java -> /etc/alternatives/java
$ ls -lrt /etc/alternatives/java #/etc/alternatives/java是ls -lrt /usr/bin/java查出来的路径
 lrwxrwxrwx. 1 root root 46 Sep 16  2015 /etc/alternatives/java -> /usr/lib/jvm/jre-1.6.0-openjdk.x86_64/bin/java
```
## 查询 crontab 定时器配置情况
```bash
crontab -l
```
## 编辑 crontab 定时器
```bash
crontab -e
# 当 crontab 调用时，错误和标准输出会写成 mail 通知你
* * * * * /opt/restart.sh
# 标准输出重定向到 /dev/null，输出到这里就找不回来了
* * * * * /opt/restart.sh > /dev/null
# 标准输出和错误输出都重定向到 /dev/null
* * * * * /opt/restart.sh > /dev/null 2>&1
```
