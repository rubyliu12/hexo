---
title: Linux cat 命令
categories:
  - Linux
tags:
  - shell
  - cat
abbrlink: be15a742
date: 2018-02-06 09:35:05
---

# cat
## 统计出现次数
```bash
# linux统计某一文件中字符串“sent ip”出现的次数：
cat nohup.log |grep "sent ip" |wc -l
```
## 清空默认文件中的内容
```bash
cat /dev/null > filename
```
