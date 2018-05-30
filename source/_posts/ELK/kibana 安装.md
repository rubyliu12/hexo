---
title: kibana安装配置
categories:
  - ELK
tags:
  - ELK
  - kibana
abbrlink: 33aa0af8
date: 2017-03-21 15:46:22
---

`Kibana` 是一个基于浏览器页面的 `Elasticsearch` 前端展示工具。`Kibana` 全部使用 `HTML` 语言和 `JavaScript` 编写的。
`Kibana` 是一个为 `Logstash` 和 `ElasticSearch` 提供的日志分析的 `Web` 接口。可使用它对日志进行高效的搜索、可视化、分析等各种操作。

### 1、配置
编辑
```sh
vi config/kibana.yml
```
<!-- more -->
按照要求修改为
```sh
elasticsearch.url: "http://192.168.56.101:9200"
```
其他部分参数不修改

如果只允许本机可以访问，在 `kibana.yml` 中加入：
```sh
host: "127.0.0.1"
```

### 2、启动

`kibana 5.2.1` 可以单独启动，命令
```sh
bin/kibana
```
后台运行
```sh
nohup bin/kibana &
```
访问
http://localhost:5601
