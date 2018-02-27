---
title: Linux sed 命令
date: 2018-02-06 09:35:05
categories: [Linux]
tags: [shell, sed]
description:
permalink: linux-shell-sed
---

# sed

## 打印指定行数
```bash
sed -n '12457431, 12457440p' tomcat/logs/catalina.out
```
