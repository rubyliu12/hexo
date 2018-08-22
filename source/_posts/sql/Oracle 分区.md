---
title: Oracle 分区
categories:
  - Oracle
tags:
  - null
abbrlink: '7150914'
date: 2018-06-05 17:01:05
---



# 自增分区

> numtoyminterval 按年、月分区

numtoyminterval(1, 'year')、numtoyminterval(1, 'month')

> numtodsinterval 按天、时、分、秒分区

numtodsinterval(1, 'day')、numtodsinterval(1, 'hour')、numtodsinterval(1, 'minute')、numtodsinterval(1, 'second')

> 例子

```sql
create table user (id number, create_dt date)
	tablespace users
	partition by range (create_dt)
	interval(numtoyminterval(1, 'month'))
	(
        partition p1 values less than (to_date('2018-06-01 00:00:00', 'yyyy-mm-dd hh24:mi:ss'))
    );
```
