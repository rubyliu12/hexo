---
title: Redis 使用场景
categories:
  - redis
tags:
  - redis
abbrlink: c77ba0b2
date: 2018-07-24 10:43:05
---



## 场景一：取最新N条数据

网站最新文章、最新评论等。使用 Redis 列表(List)集合，LPUSH 命令向 list 集合头部插入数据，LTRIM 命令对 list 集合进行修剪(trim) 

```sh
# 将一个或多个值插入到列表头部
# LPUSH key value1 [value2] 
127.0.0.1:6379> LPUSH latest.comments "foo"
(integer) 1

# 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
# LTRIM key start stop 
127.0.0.1:6379> LTRIM latest.comments 0 10
OK
```



<!-- more -->



## 场景二：排行榜应用，取 TOP N操作

场景二和场景一不同之处在于，场景一操作以时间为权重，场景二是以某一个条件为权重。比如：赞次数、评论次数等。这个时候就需要使用 Redis 有序集合(sorted set)。将需要排序的值设置为有序集合的 score，将具体的数据设置为相应的 value，每次只需要执行 ZDAA 命令即可。

```sh
# 向有序集合添加一个或多个成员，或者更新已存在成员的分数
# ZADD key score1 value1 [score2 value12] 
127.0.0.1:6379> ZADD myzset 1 "one"
```



## 场景三：计数器

Redis 命令是原子性的，可以利用 INCR、DECR 命令来构建计数器系统，其底层写入是单线程模型，并发写入会按先后顺序执行。

```sh
# 将 key 中储存的数字值增一
# INCR key
127.0.0.1:6379> INCR KEY_NAME
# 延伸
# 将 key 所储存的值加上给定的增量值（increment）
# INCRBY key increment

# 将 key 所储存的值加上给定的浮点增量值（increment）
# INCRBYFLOAT key increment

# 将 key 中储存的数字值减一
# DECR key
127.0.0.1:6379> DECR KEY_NAME 
# 延伸
# key 所储存的值减去给定的减量值（decrement）
# DECRBY key decrement
```



## 场景四：缓存

Redis 作为分布式缓存中间件，其性能优于 Memcached，且数据结构更加多样化。



## 场景五：队列

使用 Redis 列表(List) 可以构建队列系统，使用 Redis 有序集合(sorted set)甚至可以构建有优先级的队列系统。



## 场景六：Pub、Sub实时消息系统

使用 Redis 的 Pub/Sub 可以构建实时的消息系统。



## 场景七：Unique 操作

获取所有数据去重。使用 Redis 集合(Set)数据结构最合适不过了，只需要不断将数据往 set 中丢就行了，set 会自动去重。



## 场景八：精确设置过期时间

比如场景二中，score 可以设置成过期时间的时间戳，这样就可以简单的通过过期时间排序，定时清除过期数据。



...更多使用场景待挖掘