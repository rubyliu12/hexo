---
title: Oracle 使用
categories:
  - Oracle
tags:
  - 
abbrlink: db736859
date: 2018-04-08 17:01:05
---

# 修改表空间
```sql
-- 修改表空间
SELECT 'ALTER TABLE WWTEST.' || T.TABLE_NAME || ' MOVE TABLESPACE USERS;'
  FROM ALL_TABLES T
 WHERE T.OWNER = 'WX_KBT_TEST';
---- 如果有分区，使用这条语句
SELECT 'alter table ' || TABLE_NAME || ' move partition ' || PARTITION_NAME ||
       ' tablespace USERS;'
  FROM USER_TAB_PARTITIONS
 WHERE TABLE_NAME IN ('IM_WX_QX_TEMPLATE_RESULT', 'IM_WX_QX_TEMPLATE');

-- 修改索引的表空间
SELECT 'alter index ' || INDEX_NAME || ' rebuild tablespace USERS;'
  FROM ALL_INDEXES I
 WHERE I.OWNER = 'WX_KBT_TEST'
   AND I.PARTITIONED = 'YES';
---- 如果有分区，使用这条语句
SELECT 'alter index ' || P.INDEX_NAME || ' rebuild PARTITION ' ||
       P.PARTITION_NAME || ' tablespace USERS;'
  FROM USER_IND_PARTITIONS P
 WHERE P.INDEX_NAME IN ('I_IM_WX_LOG_PROCESSTYPE', 'I_IM_WX_LOG_USERNO');
```
