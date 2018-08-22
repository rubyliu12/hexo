---
title: camel-ftp 使用笔记
categories:
  - Java
tags:
  - Apache
  - camel-ftp
abbrlink: 6ae6b8a1
date: 2018-08-09 19:11:20
---



# idempotentConsumer

```java
// tmp 是一个路径：./../tmp
// 通过文件名缓存，如果缓存中已经有这个文件名，怎不再处理这个文件
.idempotentConsumer(header(Exchange.FILE_NAME),
                        FileIdempotentRepository.fileIdempotentRepository(
                                new File(tmp, "fir-智能语音-智能语音识别清单报表.txt"),
                                100 * 1024,
                                5 * 1024 * 1024))
```



# filter

```java
// 标识文件名是“智能语音清单_20180803.txt”时，才执行之后的流程
.filter(header(Exchange.FILE_NAME).isEqualTo("智能语音清单_20180803.txt"))
    
// 标识文件名不是“智能语音清单_20180803.txt”时，才执行之后的流程
.filter(header(Exchange.FILE_NAME).isNotEqualTo("智能语音清单_20180803.txt"))
```

