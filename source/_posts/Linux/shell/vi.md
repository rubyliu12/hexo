---
title: Linux vi 命令
date: 2018-02-06 09:35:05
categories: [Linux]
tags: [shell, vi]
description:
permalink: linux-shell-vi
---

# vi
linux文件格式的问题（使用unix格式）
```bash
# 查看文件格式
vi 文件名
:set ff
# 设置文件格式为unix
:set ff=unix
# 保存退出
:wq
# 也可以直接使用
dos2unix 文件名
```
