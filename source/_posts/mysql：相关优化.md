---
title: mysql：相关优化
abbrlink: bf4ad76b
date: 2016-01-21 13:55:49
tags:
 - mysql
 - 数据库
 
categories:
 - 技术
 - mysql
---
##### 1. 如何查看sql语句执行效率？ #####

使用explain函数 ：
```sql
explain select * from user;
```

或者开启 慢查询
是否开启

```sql
show variables like '%quer%';
```


log_slow_queries=off 没有开启慢查询
long_query_time 就是慢查询时间，sql语句超过这个时间会生成日志？一般来说超过3秒就算慢了