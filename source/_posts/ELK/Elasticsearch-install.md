---
title: elasticsearch安装配置
categories:
  - ELK
tags:
  - ELK
  - elaticsearch
abbrlink: 4c158d
date: 2017-03-21 15:47:36
---
### 一、安装Elasticsearch
- 下载并安装 .zip 包
```sh
wget https://artifacts.elastic.cn/downloads/elasticsearch/elasticsearch-5.0.2.zip
sha1sum elasticsearch-5.0.2.zip
unzip elasticsearch-5.0.2.zip
cd elasticsearch-5.0.2/
```
- 下载并安装 .tar.gz 包
```sh
wget https://artifacts.elastic.cn/downloads/elasticsearch/elasticsearch-5.0.2.tar.gz
sha1sum elasticsearch-5.0.2.tar.gz
tar -zxvf elasticsearch-5.0.2.tar.gz
cd elasticsearch-5.0.2/
```
<!-- more -->
### 二、运行 Elasticsearch
```
./bin/elasticsearch
```
Window 系统也是一样。默认的，Elasticsearch 运行在前台，并且打印日志，可以通过 ctrl+c 来停止它的运行。Windows 系统还可以通过 elasticsearch-service.bat 命令来将 Elasticsearch 注册为一个服务。  
你可以在浏览器打开 localhost:9200，来查看 Elasticsearch 是否运行。  
日志输出可以通过 -q 或者 -quiet 选项来禁止。  
将 Elasticsearch 当做守护进程来运行
```
./bin/elasticsearch -d -p pid
```
日志信息记录在 $ES_HOME/logs/，proces id 会记录在 pid 文件。  
停止 Elasticsearch 使用如下命令：
```
kill 'cat pid'
```

### 三、配置 Elasticsearch
Elasticsearch 默认从 $ES_HOME/config/elasticsearch.yml 加载配置文件。  
配置文件中的配置都可以通过命令来设定，使用 -E 语法：
```
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```
如果值中带有空格，在值必须使用""包囊起来。  
一般，集群层面的配置（例如：cluster.name）应该添加到 elasticsearch.yml 文件，近点层面的设置（例如：node.name）可以通过命令来设置。

#### 3.1 重要的系统配置  
启动 `elasticsearch` 时候遇到 `nofile`、`nproc`、`jvm` 等报错

#### 3.2 设置 JVM 参数
首选的设置JVM参数的方法是通过 config/jvm.options 配置文件（使用 .zip 或者 .tar.gz 安装包时），/etc/elasticsearch/jvm.options（当使用 Debian 或者 RPM 安装包时）。JVM 参数必须以 - 开始，你可以添加自定义JVM标识并且把这个配置上传到你的版本控制系统。

默认的 Elasticsearch 设置的 JVM 内存大小为2GB，当向生产环境部署时要确定 Elasticsearch 有足够的内存可用。好的规则是：

- 设置-Xms与-Xmx相等
- 注意太大的内存容易产生长的垃圾回收停顿
- -Xmx不要超过你物理内存的50%
- 不要设置-Xmx超过JVM用来压缩对象指针的大小，精确的临界值不是固定的，但是接近于32GB。  

下面是设置JVM内存大小的例子：
```
-Xms2g 
-Xmx2g
```

#### 3.3 禁止内存交换
大多数操作系统会用尽可能多的内存用于文件系统缓存和及早换出无用的应用内存。这可能导致一部分JVM内存被交换到硬盘上。

这种内存交换非常不利于性能和节点的稳定性。应该竭尽所能来避免这种情况。它能引起垃圾回收持续长达数分钟而不是几毫秒并且能导致节点响应缓慢甚至与集群失去联系。

这里有三种方法来禁止内存交换：  

##### 启用 bootstrap.memory_lock  
通过使用 Linux/Unix 系统的 mlockall 或者 Windows 系统的 VirtualLock 尝试锁定 RAM 中的进程地址空间，防止任何 Elasticsearch 内存被交换。可以通过向 config/elasticsearch.yml 文件中添加以下语句来实现：
```
bootstrap.memory_lock: true
```
警告：mlockall 如果尝试分配超过了可用的内存，可能会引起 JVM 或者 shell session 退出。在启动E lasticsearch 后，你可以检查下设置是否生效了，可以通过检查下面请求响应中的 mlockall 值：
```
GET _nodes?filter_path=**.mlockall
```
如果你看到 mlockall 是 false，那说明禁止内存交换失败了。并且日志里会看到类似下面的话：Unable to lock JVM Memory.

最可能的原因是，在Linux/Unix操作系统，用户运行的Elasticsearch没有锁定内存的权限。可以通过下面的方法来解决：

使用 .zip 和 .tar.gz 这种安装包时，在 /etc/security/limits.conf 文件中设置 memlock 为 unlimited。

其他方式就不翻译了。

RPM and Debian
Set MAX_LOCKED_MEMORY to unlimited in the system configuration file (or see below for systems using systemd).
Systems using systemd
Set LimitMEMLOCK to infinity in the systemd configuration.

还有一种 mlockall 可能失败的情况是临时文件夹（通常是 /tmp ）挂载时设定了 noexec 选项。这种情况可以通过指定一个新的临时文件夹来解决，使用 ES_JAVA_OPTS 环境变量：
```
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
./bin/elasticsearch
```
或者在 jvm.options 配置文件里设置。  

##### 禁止所有文件交换  
第二种禁止内存交换的方式是完全禁止交换。通常 Elasticsearch 是一台服务器唯一运行的服务，它的内存使用被JVM控制着，所以没有必要允许交换。

在 Linux 系统你可以通过运行以下代码禁止内存交换：
```
sudo swapoff -a
```
如果要永久禁用，则需要编辑 /etc/fstab 文件，然后注释掉所有包含 swap 的行。

在 Windows 系统可以通过系统属性 → 高级系统设置 → 性能 → 高级选项卡 → 虚拟内存 然后设置为无分页文件。（Windows 7）

#### 3.4 文件描述符
这个设置只针对 Linux 和 macOS 操作系统，如果运行在 Windows 系统则可以安全的被忽略。

Elasticsearch 使用了大量的文件描述符或者文件句柄。文件描述符将要被用完时会导致灾难性的后果，并且非常可能引起数据丢失。确保增加运行 Elasticsearch 的用户打开文件描述符的数量至少为65,536或者更高。

对于 .zip 和 .tar.gz 安装包，在启动 Elasticsearch 前以root身份设置ulimit -n 65536，或者修改 /etc/security/limits.conf 文件，设置 nofile 参数为65536或更高。

RPM 和 Debian 的安装包已经设置了默认最大文件描述符数为65536，不需要额外配置。

你可以检查每个节点的 max_file_descriptors 配置情况：
```
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```

#### 3.5 虚拟内存
Elasticsearch 默认使用一个 hybrid mmapfs/niofs 来存储它的索引。操作系统默认限制的内存映射数是比较低的，可能会引起内存溢出异常。

在 Linux，你可以用 root 身份通过以下命令来增加这个限制：
```
sysctl -w vm.max_map_count=262144
```
要想永久的增加 vm.max_map_count 设置，需要编辑 /etc/sysctl.conf 文件。重启后通过：
```
sysctl vm.max_map_count
```
来检验设置是否生效

RPM 和 Debian安装包将会自动设置这个配置。不需要额外的操作。

#### 3.6 线程数
Elasticsearch 使用多个线程池来进行不同类型的操作。当需要时能够创建新线程是很重要的。确保 Elasticsearch 用户能创建的线程数最少为2048个。

可以在启动前通过设置 ulimit -u 2048，或者在 /etc/security/limits.conf 文件里设置 nproc 为2048。
