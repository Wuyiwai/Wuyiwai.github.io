---
title: SQL常见命令
date: 2019-10-21 02:16:36
tags: SQL
categories: SQL
---

> 记录常见的SQL命令

<!--more-->

1. 查询表索引
```
MySQL
show index from table_name（表名）

PgSQL
select * from pg_indexes where tablename='tbname';
```

2. 查看表字段
```
MySQL
show create table table_name;

PgSQL
```