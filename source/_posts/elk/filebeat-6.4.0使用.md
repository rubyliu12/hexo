---
title: filebeat-6.4.0 使用
categories:
  - elk
tags:
  - filebeat
abbrlink: 9cf2c75c
date: 2018-08-30 09:35:05
---



# 管理多行消息

```yml
filebeat.inputs:
- type: log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/logs/*.log
```

