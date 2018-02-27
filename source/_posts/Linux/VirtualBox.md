---
title: VirtualBox 虚拟机安装 CentOS
date: 2017-07-07 08:54:46
categories: [Linux]
tags: [VirtualBox]
description:
permalink: virtualbox-centos
---
# VirtualBox 安装 CentOS 7


### 安装
> 虚拟机安装CentOS步骤省略。推荐安装minimal版本

### 配置网络

> ***CentOS7*** 中已经取消了 ***ifconfig***，用 ***nmcli*** 进行了代替，服务管理也升级为 ***systemd***。所以之前在6.x版本上的网络配置操作在7.x上行不通了。
下面介绍一下在CentOS7.x上进行网络配置的方法。