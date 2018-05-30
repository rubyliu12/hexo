---
title: elasticsearch踩坑
categories:
  - ELK
tags:
  - ELK
  - elasticsearch
abbrlink: 19540ce1
date: 2017-03-21 15:48:48
---
- 问题一  

max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]  
配置 `/etc/sysctl.conf`
```sh
$ vi /etc/sysctl.conf
vm.max_map_count=262144
```


<!-- more -->

生效

```sh
$ sysctl -p
```
验证
```sh
$ sysctl -a|grep vm.max_map_count
```

- 问题二  

max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]  
max number of threads [1024] for user [XXX] is too low，increase to least [2048]  
1、修改 `nofile` 时候，需要在 `/etc/sysctl.conf` 和 `/etc/security/limits.conf` 配置，其中 `/etc/sysctl.conf` 中配置的值需要比 `/etc/security/limits.conf` 的值大  
配置 `/etc/sysctl.conf`
```sh
$ vi /etc/sysctl.conf 
fs.file-max = 512000
```
生效
```sh
$ sysctl -p
```
配置 `/etc/security/limits.conf`
```sh
$ vi /etc/security/limits.conf
elasticsearch - nofile 65536
```
或者
```sh
* - nofile 65536
```

2、修改 `nofile` 和 `nproc`  
`CentOS 7` 的 `nproc` 的修改在 `/etc/security/limits.d/20-nproc.conf`，为了方便在 `/etc/security/limits.conf` 和 `/etc/security/limits.d/20-nproc.conf` 文件里面同时都添加了  
配置 `/etc/security/limits.conf`
```sh
$ vi /etc/security/limits.conf
* - nproc 65536
```
配置 `/etc/security/limits.d/20-nproc.conf`
```sh
$ vi /etc/security/limits.d/20-nproc.conf
* - nofile 65536
* - nproc 65536
```
重启系统，验证
```sh
$ ulimit -Hn
```

总结：
```sh
$ vi /etc/sysctl.conf
vm.max_map_count=262144
fs.file-max=512000

$ vi /etc/security/limits.conf
elasticsearch - nofile 65536
* - nproc 65536

$ vi /etc/security/limits.d/20-nproc.conf
* - nofile 65536
* - nproc 65536
```
