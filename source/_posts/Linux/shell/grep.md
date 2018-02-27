---
title: Linux grep 命令
date: 2018-02-06 09:35:05
categories: [Linux]
tags: [shell, grep]
description:
permalink: linux-shell-grep
---

# grep
## 统计出现次数
```bash
grep 'sign.*time.*openid' access_p80_weixinv3.log | awk '{a[$1]++}END{for(i in a)print a[i]"\t"i}' | sort -n
```

##  统计某些字符出现次数
```bash
grep 'sign.*time.*openid' access_p80_weixinv3.log | awk '{a[$1]++}END{for(i in a)print a[i]"\t"i}' | sort -n
```
4       183.3.234.45
10      183.3.234.57
20      183.3.234.58

## 查询前后日志
grep -5 'parttern' inputfile //打印匹配行的前后5行
grep -C 5 'parttern' inputfile //打印匹配行的前后5行
grep -A 5 'parttern' inputfile //打印匹配行的后5行
grep -B 5 'parttern' inputfile //打印匹配行的前5行
