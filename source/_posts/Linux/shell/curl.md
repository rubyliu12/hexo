---
title: Linux curl 命令
categories:
  - Linux
tags:
  - shell
  - curl
abbrlink: aea02818
date: 2018-02-06 09:35:05
---

# curl
# GET 请求
```bash
curl http://www.baidu.com?q=xxxx
```
## POST 请求
```bash
curl http://10.121.2.35/weixin_v3/w?urlToken=wxmanager -XPOST -d "send message..."
```
## 发送 POST JSON 请求
```bash
curl -H "Content-Type: application/json" -X POST  --data '{"data":"1"}' http://127.0.0.1/
```
## 使用代理
```bash
curl -x 192.168.2.5:3128 http://www.baidu.com?q=xxxx
```
## 输出请求信息
```bash
# http_code
# time_connect
# time_starttransfer
# time_total
# size_download
# speed_download
# 参考地址：
curl http://www.baidu.com?q=xxxx -so /dev/null -w "%{http_code}\t%{time_connect}\t%{time_starttransfer}\t%{time_total}\t%{size_download}\t%{speed_download}\t"
```
