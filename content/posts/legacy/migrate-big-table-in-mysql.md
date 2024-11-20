---
author: ["Me"]
title: 'Migrate Big Table in MySQL'
date: 2024-04-11T16:23:32+08:00
categories: ["Database"]
tags: ["MySQL", "Database Migration", "Big Data"]
draft: false
---

Migrate MySQL big table(50 Million) rows using

 `insert into xxx select * from yyy`

and it said 

```
ERROR 1206 (HY000): The total number of locks exceeds the lock table size
```

It's due to MySQL MVCC control, the rows were in memory and exceeds the limit. 

You can use this command to quickly find the limit:

```
SELECT @@innodb_buffer_pool_size/1024/1024/1024;
```

For me, that's 0.125. So I changed it by this command:

```
SET GLOBAL innodb_buffer_pool_size = 1073741824;
```

10x the original size, so everything works fine. 

Or You can do it more safely, lock the table and there's no need to cache the rows to control version, just as below:

```
SET @@AUTOCOMMIT=0;
LOCK TABLES xxx WRITE, yyy READ;
INSERT INTO xxx
SELECT * FROM yyy
GROUP BY date;
UNLOCK TABLES;
```

## References

[InnoDB バッファープールをオンラインで変更する](https://qiita.com/mita2/items/8fd915164f0851c96e54)

[The total number of locks exceeds the lock table size](https://stackoverflow.com/questions/6901108/the-total-number-of-locks-exceeds-the-lock-table-size)
