---
title: 阿里巴巴druid连接池配置介绍
categories: Java
tags:
  - druid
  - DataSource
abbrlink: facb8eb2
date: 2017-05-23 22:07:42
---
druid连接池配置

- filters=stat
属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：
监控统计用的 filter:stat
日志用的 filter:log4j
防御sql注入的 filter:wall
- initialSize=5
初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时，默认：0
- minIdle=5
最小连接池数量
- maxActive=30
最大连接池数量，默认：8
- maxWait=60000
获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
<!-- more -->
- poolPreparedStatements=true
是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。默认：false
- maxOpenPreparedStatements=20
要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100。默认：-1
- timeBetweenEvictionRunsMillis=90000
有两个含义：1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
- minEvictableIdleTimeMillis=1800000
连接保持空闲而不被驱逐的最长时间（单位：ms）
- validationQuery=SELECT 1 FROM DUAL
- testWhileIdle=true
建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。默认：false
- testOnBorrow=false
申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。默认：true
- testOnReturn=false
归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。默认：false
