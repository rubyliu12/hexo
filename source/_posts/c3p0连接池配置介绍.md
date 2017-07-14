---
title: c3p0连接池配置介绍
date: 2017-05-23 21:58:36
categories: [Java]
tags: [c3p0, DataSource]
description:
permalink:
---
com.mchange.v2.c3p0.ComboPooledDataSource参数设置

- acquireIncrement -> 1
当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3
- acquireRetryAttempts -> 30
定义在从数据库获取新连接失败后重复尝试的次数。Default: 30
- acquireRetryDelay -> 1000
两次连接中间隔时间，单位毫秒。Default: 1000
- autoCommitOnClose -> false
连接关闭时默认将所有未提交的操作回滚。Default: false
- automaticTestTable -> null
c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么 属性 preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试使用。Default: null
<!-- more -->
- breakAfterAcquireFailure -> false
获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效保留，并在下次调用getConnection() 的时候继续尝试获取连接。如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。Default: false
- checkoutTimeout -> 0
当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出SQLException,如设为0则无限期等待。单位毫秒。efault: 0
- connectionCustomizerClassName -> null
定制管理Connection的生命周期
- connectionTesterClassName -> com.mchange.v2.c3p0.impl.DefaultConnectionTester
通过实现ConnectionTester或QueryConnectionTester的类来测试连接。类名需制定全路径。Default: com.mchange.v2.c3p0.impl.DefaultConnectionTester
- dataSourceName -> 1hgf49x9esziub21tsqy1p|17b6178
- unreturnedConnectionTimeout -> 0
配置debug和回收Connection：单位s
- debugUnreturnedConnectionStackTraces -> false
配置debug和回收Connection
- description -> null
- driverClass -> oracle.jdbc.driver.OracleDriver
- factoryClassLocation -> null
指定c3p0 libraries的路径，如果（通常都是这样）在本地即可获得那么无需设置，默认null即可。Default: null
- forceIgnoreUnresolvedTransactions -> false
Strongly disrecommended. Setting this to true may lead to subtle and bizarre bugs. 文档原文）作者强烈建议不使用的一个属性
- identityToken -> 1hgf49x9esziub21tsqy1p|17b6178
- idleConnectionTestPeriod -> 60
每60秒检查所有连接池中的空闲连接。Default: 0
- initialPoolSize -> 5
初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3
- jdbcUrl -> jdbc:oracle:thin:@localhost:1521:orcl
- maxAdministrativeTaskTime -> 0
- maxConnectionAge -> 0
管理连接池的大小和连接的生存时间：单位s
- maxIdleTime -> 60
最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0
- maxIdleTimeExcessConnections -> 0
default : 0，单位 s 。这个配置主要是为了减轻连接池的负载，比如连接池中连接数因为某次数据访问高峰导致创建了很多数据连接，但是后面的时间段需要的数据库连接数很少，则此时连接池完全没有必要维护那么多的连接，所以有必要将断开丢弃掉一些连接来减轻负载，必须小于maxIdleTime。配置不为0，则会将连接池中的连接数量保持到minPoolSize
- maxPoolSize -> 20
连接池中保留的最大连接数。Default: 15
- maxStatements -> 0
JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个 connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。如果maxStatements与 maxStatementsPerConnection均为0，则缓存被关闭。Default: 0
- maxStatementsPerConnection -> 0
maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0
- minPoolSize -> 5
初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3
- numHelperThreads -> 3
c3p0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能 通过多线程实现多个操作同时被执行。Default: 3
- numThreadsAwaitingCheckoutDefaultUser -> 0
- preferredTestQuery -> null
定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个一显著提高测试速度。注意：测试的表必须在初始数据源的时候就存在。Default: null
- properties -> {user=****** password=******}
- propertyCycle -> 0
用户修改系统配置参数执行前最多等待300秒。Default: 0
- testConnectionOnCheckin -> false
如果设为true那么在取得连接的同时将校验连接的有效性。Default: false
- testConnectionOnCheckout -> false
因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的 时候都将校验其有效性。建议使用 idleConnectionTestPeriod或automaticTestTable 等方法来提升连接测试的性能。Default: false
- usesTraditionalReflectiveProxies -> false
