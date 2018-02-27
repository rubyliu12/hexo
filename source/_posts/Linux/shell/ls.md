---
title: Linux ls 命令
date: 2018-02-06 09:35:05
categories: [Linux]
tags: [shell, ls]
description:
permalink: linux-shell-ls
---

# ls
```bash
# 统计文件夹中文件数
ls -l |grep "^-"|wc -l
```
## 保留最新的1个文件夹，删除其他文件夹（使用-d）
```bash
ls -t -d /home/txrd/tisson-web-open/logs/undertow-* | tail -n +2 | xargs rm -rf
```
