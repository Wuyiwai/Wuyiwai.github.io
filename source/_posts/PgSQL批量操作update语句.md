---
title: PgSQL批量操作update语句
date: 2019-10-09 21:53:45
tags: PostgreSQL
categories: PostgreSQL
---
> 参考链接
> 1. [stackoverflow](https://stackoverflow.com/questions/18797608/update-multiple-rows-in-same-query-using-postgresql)

##### 想要达到的效果,合并对同一个表的多次update操作
```
类似于
UPDATE table 
SET 
 column_a = 1 where column_b = '123',
 column_a = 2 where column_b = '345'
```

<!--more-->

##### 更新单列
```
update test as t set
    column_a = c.column_a
from (values
    ('123', 1),
    ('345', 2)  
) as c(column_b, column_a) 
where c.column_b = t.column_b;
```

##### 更新多列
```
update test as t set
    column_a = c.column_a,
    column_c = c.column_c
from (values
    ('123', 1, '---'),
    ('345', 2, '+++')  
) as c(column_b, column_a, column_c) 
where c.column_b = t.column_b;
```