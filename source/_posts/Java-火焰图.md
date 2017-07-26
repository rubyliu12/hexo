---
title: Brendan Gregg 的 FlameGraph 火焰图
date: 2017-07-15 21:58:36
categories: [Java]
tags: [lightweight-java-profiler, FlameGraph]
description:
permalink: flameGraph
---
# 下载 lightweight-java-profiler 采样栈信息

`https://code.google.com/archive/p/lightweight-java-profiler/source/default/source` 下载 `lightweight-java-profiler` 工具

1. 修改
```bash
# 第4行 BITS?=32 改成
BITS?=64
# 第49行 INCLUDES=-I$(JAVA_HOME)/$(HEADERS) -I$(JAVA_HOME)/$(HEADERS)/$(UNAME) 改成
INCLUDES=-I$(JAVA_HOME)/$(HEADERS) -I$(JAVA_HOME)/$(HEADERS)/$(UNAME) -I/usr/include/x86_64-linux-gnu
```

<!-- more -->
2. 编译
```bash
make all
# yum -y update gcc
# yum -y install gcc+ gcc-c++
# 报错的话，要修改display.cc 第 22 行，fprintf(file_, "%" PRIdPTR" ", traces[i].count);
```

# 下载 FlameGraph 火焰图生成工具
```bash
git clone http://github.com/brendangregg/FlameGraph
```

# 使用 Profiler 工具生成 trace.txt 文件
```bash
# tomcat 中
JAVA_OPTS="-server -agentpath:/home/yl/lightweight-java-profiler/build-64/liblagent.so"

# java jar
java -agentpath:/home/yl/lightweight-java-profiler/build-64/liblagent.so -jar xxx.jar
```
> 注意：trace 不是实时写入，而是在应用 shutdown 的时候才写入的，别 kill 应用，否则 trace 里面什么都没有

另外有几个参数可在编译时修改，都在global.h文件中。首先是采样的频率，缺省是100次 每秒；另外是最大采样的线程栈，缺省3000，超过3000就忽略（对于复杂的应用明显不够） ；最后是栈的深度，缺省是128（对于调用层次深的应用调大）。

# 将 trace.txt 文件转换为 trace.svg 格式的火焰图
```bash
cd FlameGraph
./stackcollapse-ljp.awk < ../traces.txt | ./flamegraph.pl --colors=java --title="traces title" > ../traces.svg
# yum install perl
# 一些参数说明
--title       # change title text
--width       # width of image (default 1200)
--height      # height of each frame (default 16)
--minwidth    # omit smaller functions (default 0.1 pixels)
--fonttype    # font type (default "Verdana")
--fontsize    # font size (default 12)
--countname   # count type label (default "samples")
--nametype    # name type label (default "Function:")
--colors      # set color palette. choices are: hot (default), mem, io,
              # wakeup, chain, java, js, perl, red, green, blue, aqua,
              # yellow, purple, orange
--hash        # colors are keyed by function name hash
--cp          # use consistent palette (palette.map)
--reverse     # generate stack-reversed flame graph
--inverted    # icicle graph
--negate      # switch differential hues (blue<->red)
--help        # this message

eg,
./flamegraph.pl --title="Flame Graph: malloc()" trace.txt > graph.svg
```
用浏览器打开生成的 traces.svg 火焰图文件

默认：samples
--countname=pages
1 pages = 4 kbyte
--countname=bytes
