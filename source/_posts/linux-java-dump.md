---
title: linux dump内存快照等
categories:
  - linux
tags:
  - linux, Java
permalink: linux-java-dump
date: 2017-07-07 08:50:17
description:
---
# kill之前先dump
每次线上环境一出问题，大家就慌了，

通常最直接的办法回滚重启，以减少故障时间，

这样现场就被破坏了，要想事后查问题就麻烦了，

有些问题必须在线上的大压力下才会发生，

线下测试环境很难重现，

不太可能让开发或Appops在重启前，

先手工将出错现场所有数据备份一下，

所以最好在kill脚本之前调用dump，

进行自动备份，这样就不会有人为疏忽。

# dump脚本示例
<!-- more -->
```bash
JAVA_HOME=/usr/java
OUTPUT_HOME=~/output
DEPLOY_HOME=`dirname $0`
HOST_NAME=`hostname`

DUMP_PIDS=`ps  --no-heading -C java -f --width 1000 | grep "$DEPLOY_HOME" |awk '{print $2}'`
if [ -z "$DUMP_PIDS" ]; then
    echo "The server $HOST_NAME is not started!"
    exit 1;
fi

DUMP_ROOT=$OUTPUT_HOME/dump
if [ ! -d $DUMP_ROOT ]; then
    mkdir $DUMP_ROOT
fi

DUMP_DATE=`date +%Y%m%d%H%M%S`
DUMP_DIR=$DUMP_ROOT/dump-$DUMP_DATE
if [ ! -d $DUMP_DIR ]; then
    mkdir $DUMP_DIR
fi

echo -e "Dumping the server $HOST_NAME ...\c"
for PID in $DUMP_PIDS ; do
    $JAVA_HOME/bin/jstack $PID > $DUMP_DIR/jstack-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jinfo $PID > $DUMP_DIR/jinfo-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jstat -gcutil $PID > $DUMP_DIR/jstat-gcutil-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jstat -gccapacity $PID > $DUMP_DIR/jstat-gccapacity-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jmap $PID > $DUMP_DIR/jmap-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jmap -heap $PID > $DUMP_DIR/jmap-heap-$PID.dump 2>&1
    echo -e ".\c"
    $JAVA_HOME/bin/jmap -histo $PID > $DUMP_DIR/jmap-histo-$PID.dump 2>&1
    echo -e ".\c"
    if [ -r /usr/sbin/lsof ]; then
        /usr/sbin/lsof -p $PID > $DUMP_DIR/lsof-$PID.dump
        echo -e ".\c"
    fi
done
if [ -r /usr/bin/sar ]; then
/usr/bin/sar > $DUMP_DIR/sar.dump
echo -e ".\c"
fi
if [ -r /usr/bin/uptime ]; then
/usr/bin/uptime > $DUMP_DIR/uptime.dump
echo -e ".\c"
fi
if [ -r /usr/bin/free ]; then
/usr/bin/free -t > $DUMP_DIR/free.dump
echo -e ".\c"
fi
if [ -r /usr/bin/vmstat ]; then
/usr/bin/vmstat > $DUMP_DIR/vmstat.dump
echo -e ".\c"
fi
if [ -r /usr/bin/mpstat ]; then
/usr/bin/mpstat > $DUMP_DIR/mpstat.dump
echo -e ".\c"
fi
if [ -r /usr/bin/iostat ]; then
/usr/bin/iostat > $DUMP_DIR/iostat.dump
echo -e ".\c"
fi
if [ -r /bin/netstat ]; then
/bin/netstat > $DUMP_DIR/netstat.dump
echo -e ".\c"
fi
echo "OK!"
```
具体语句示例
```bash
###########################################################################################
/usr/java/jdk1.6.0_45/bin/jstack 30965 > /opt/wxOpenServer/jstack-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jinfo 30965 > /opt/wxOpenServer/jinfo-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jstat -gcutil 30965 > /opt/wxOpenServer/jstat-gcutil-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jstat -gccapacity 30965 > /opt/wxOpenServer/jstat-gccapacity-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jmap 30965 > /opt/wxOpenServer/jmap-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jmap -heap 30965 > /opt/wxOpenServer/jmap-heap-30965.dump 2>&1;
/usr/java/jdk1.6.0_45/bin/jmap -histo 30965 > /opt/wxOpenServer/jmap-histo-30965.dump 2>&1;
/usr/sbin/lsof -p 30965 > /opt/wxOpenServer/lsof-30965.dump;
###########################################################################################
```
lsof输出各列信息的意义如下：
COMMAND：进程的名称

PID：进程标识符

USER：进程所有者

FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等

TYPE：文件类型，如DIR、REG等

DEVICE：指定磁盘的名称

SIZE：文件的大小

NODE：索引节点（文件在磁盘上的标识）

NAME：打开文件的确切名称

lsof `which httpd` //那个进程在使用apache的可执行文件

lsof /etc/passwd //那个进程在占用/etc/passwd

lsof /dev/hda6 //那个进程在占用hda6

lsof /dev/cdrom //那个进程在占用光驱

lsof -c sendmail //查看sendmail进程的文件使用情况

lsof -c courier -u ^zahn //显示出那些文件被以courier打头的进程打开，但是并不属于用户zahn

lsof -p 30297 //显示那些文件被pid为30297的进程打开

lsof -D /tmp 显示所有在/tmp文件夹中打开的instance和文件的进程。但是symbol文件并不在列

lsof -u1000 //查看uid是100的用户的进程的文件使用情况

lsof -utony //查看用户tony的进程的文件使用情况

lsof -u^tony //查看不是用户tony的进程的文件使用情况(^是取反的意思)

lsof -i //显示所有打开的端口

lsof -i:80 //显示所有打开80端口的进程

lsof -i -U //显示所有打开的端口和UNIX domain文件

lsof -i UDP@[url]www.akadia.com:123 //显示那些进程打开了到www.akadia.com的UDP的123(ntp)端口的链接

lsof -i tcp@ohaha.ks.edu.tw:ftp -r //不断查看目前ftp连接的情况(-r，lsof会永远不断的执行，直到收到中断信号,+r，lsof会一直执行，直到没有档案被显示,缺省是15s刷新)

lsof -i tcp@ohaha.ks.edu.tw:ftp -n //lsof -n 不将IP转换为hostname，缺省是不加上-n参数
