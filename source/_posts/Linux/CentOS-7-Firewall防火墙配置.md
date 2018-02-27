---
title: CentOS 7 Firewall防火墙配置
date: 2017-03-21 15:34:07
categories: [Linux]
tags: [Firewall, iptables]
description:
permalink: centos-firewall
---

### 一、firewall介绍
`CentOS 7`中防火墙是一个非常的强大的功能，对`CentOS 6.5`中的`iptables`防火墙进行了升级了。


### 二、firewall配置
#### 1、系统配置目录
```sh
/usr/lib/firewalld/services
```
目录中存放定义好的网络服务和端口参数，系统参数，不能修改。

#### 2、用户配置目录
```sh
/etc/firewalld/
```
<!-- more -->
#### 3、如何自定义添加端口
用户可以通过修改配置文件的方式添加端口，也可以通过命令的方式添加端口，注意，修改的内容会在`/etc/firewalld/`目录下的配置文件中还体现。  

- 3.1、命令的方式
```sh
firewall-cmd --permanent --add-port=9527/tcp
```
参数介绍：  
1、firewall-cmd：是Linux提供的操作`firewall`的一个工具  
2、--permanent：表示设置为持久  
3、--add-port：标识添加的端口  

另外，`firewall` 中有 `Zone` 的概念，可以将具体的端口制定到具体的 `zone` 配置文件中。

例如：添加 `8010` 端口
```sh
firewall-cmd --zone=public --permanent --add-port=8010/tcp
```
`--zone=public`指定的zone为public  

添加规则
```sh
$ firewall-cmd --permanent \ 
--zone=public \ 
--add-rich-rule="rule family="ipv4" source \ 
address="192.168.0.4/24" \ 
service name="http" accept"
```
删除上面配置的规则
```sh
$ firewall-cmd --permanent \ 
--zone=public \ 
--remove-rich-rule="rule family="ipv4" source \ 
address="192.168.0.4/24" \ 
service name="http" accept"
```

> You can check if the port has actually be opened by running  
> firewall-cmd --zone=<zone> --query-port=80/tcp  
> firewall-cmd --zone=<zone> --query-service=http


- 3.2、文件的方式
```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <!-- 放通指定ip，指定端口、协议 -->
  <rule family="ipv4">
    <source address="192.168.56.101"/>
    <port protocol="udp" port="555"/>
    <accept/>
  </rule>
  <rule family="ipv4">
    <source address="192.168.56.101"/>
    <port protocol="tcp" port="9200-9300"/>
    <accept/>
  </rule>
  <!-- 放通任意ip访问服务器的8081端口 -->
  <rule family="ipv4">
    <port protocol="tcp" port="8081"/>
    <accept/>
  </rule>
</zone>
```
上述的一个配置文件可以很好的看出：
```
1、添加需要的规则，开放通源ip为192.168.56.101，端口555，协议tcp
2、开放通源ip为192.168.56.101，端口9200-9300，协议tcp
3、开放通源ip为任意，端口8081，协议tcp
```

### 三、firewall常用命令
常用命令介绍
```sh
firewall-cmd --state                           ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                       ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                 ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口 
iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
man firewall-cmd                               ##查看帮助
```

重启、关闭、开启`firewalld`服务
```sh
systemctl restart firewalld #重启
systemctl start firewalld #开启
systemctl stop firewalld #关闭
```

### 四、CentOS切换为iptables防火墙
本节略
