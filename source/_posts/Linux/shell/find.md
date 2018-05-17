---
title: Linux find 命令
categories:
  - Linux
tags:
  - shell
  - find
abbrlink: 7b744fec
date: 2018-02-06 09:35:05
---

# find
## 查找文件并删除
```bash
# 删除指定时间前的 jpg 文件
find /opt/upload/ -mtime +30 -type f -name *.jpg -exec rm -f {} \;

# 删除隐藏文件（-fr）
find /opt/upload/ -type d -name .svn -exec rm -fr {} \;

# 删除30天前的文件
find /opt/upload -mtime +30 -type f -name "*.jpg" -exec rm -rf {} \;
```

## 打印指定时间前的文件
```bash
find project/upload/ -mtime +30 -type f -print
```
