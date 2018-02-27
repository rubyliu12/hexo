---
title: xampp 安装 memcache 监控
date: 2017-07-07 08:43:05
categories: [memcache]
tags: [xampp, memcache, php]
description:
permalink: xampp-memcache
---
# xampp安装memcache监控

## 现在 memadmin
下载[memadmin](https://github.com/foreveryang321/memadmin)，解压到`xampp/htdocs`下

## 查看 php 版本信息
```php
<?php
    echo phpinfo();
?>
```
<!-- more -->
## 下载 php-memcache.dll
根据查看到的`php`的信息，在[php-memcache](http://pecl.php.net/package/memcache/3.0.8/windows)下载对应本版的`Thread Safe`，里面包含了`php-memcache.dll`。

解压下载包，然后把`php-memcache.dll`拷贝到`xampp/php/ext`下

在`xampp/php/php.ini`最后添加以下内容
```
;php-memcached
extension=php_memcache.dll

[Memcache]
memcache.allow_failover = 1
memcache.max_failover_attempts=20
memcache.chunk_size =8192
memcache.default_port = 11211
```

## 访问 memadmin
http://host:port/memadmin
