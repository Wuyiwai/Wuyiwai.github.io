---
title: Mysql Explain详解
date: 2020-05-28 17:45:09
updateDate: 2020-05-28 17:45:09
tags: MySQL
categories: MySQL
---

> 概要: 详细解释Mysql Explain执行计划

<!--more-->

key | value | desc |
:---:|:---:|---|
id | 表示查询中执行select子句或者操作表的顺序 | id越大,执行优先级越高;id相同,按优化器给出顺序从上往下执行
select_type | 表示 select 查询的类型 | - |
- | SIMPLE | 简单查询,不含子查询或union等操作 |
- | PRIMARY | 含有任何复杂的字部分则标记为PRIMARY | 
- | SUBQUERY | 含有子查询 | 
- | DERIVED | 包含在from子句中的子查询的select | 
- | UNION | union后面又出现select;若union包含在from子句的子查询,外层select会被标记为DERIVED
- | UNION RESULT | 代表从union的临时表中读取数据,如果table列数值如<union1,4>,代表用id=1 和 id=4 的结果进行union操作 |
table | 查询的表名 | - |
partitions | 匹配到的分区信息 | 非分区值为NULL |
type | 单表的访问方法 | 性能: system > const > eq_ref > ref > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
- | system | 当表仅有一行记录时(系统表)，数据量很少，往往不需要进行磁盘IO，速度非常快 |
- | const | 表示查询时命中 primary key 主键或者 unique 唯一索引，或者被连接的部分是一个常量(const)值。这类扫描效率极高，返回数据量少，速度非常快。|
- | eq_ref | 查询时命中主键primary key 或者 unique key索引 |
- | ref | 区别于eq_ref ，ref表示使用非唯一性索引，会找到很多个符合条件的行 |
- | ref_or_null | 这种连接类型类似于 ref，区别在于 MySQL会额外搜索包含NULL值的行 |
- | index_merge | 使用了索引合并优化方法，查询使用了两个以上的索引。|
- | unique_subquery | 对于以下形式的某些IN子查询，此类型替换eq_ref <br> `value IN (SELECT primary_key FROM single_table WHERE some_expr)` <br> unique_subquery只是一个索引查找 function，完全替换子查询以提高效率。|
- | index_subquery | 此连接类型与unique_subquery类似。它取代了IN子查询，但它适用于以下形式的子查询中的非唯一索引：<br> `value IN (SELECT key_column FROM single_table WHERE some_expr)` |
- | range | 使用索引选择行，仅检索给定范围内的行. 简单点说就是针对一个有索引的字段，给定范围检索数据。在where语句中使用 bettween...and、<、>、<=、in 等条件查询 type 都是 range。 |
- | index | Index 与ALL 其实都是读全表，区别在于index是遍历索引树读取，而ALL是从硬盘中读取。|
- | ALL | 将遍历全表以找到匹配的行，性能最差。|
possible_keys | - | 表示在MySQL中通过哪些索引，能让我们在表中找到想要的记录，一旦查询涉及到的某个字段上存在索引，则索引将被列出，但这个索引并不定一会是最终查询数据时所被用到的索引。|
key | 区别于possible_keys，key是查询中实际使用到的索引，若没有使用索引，显示为NULL | 当 type 为 index_merge 时，可能会显示多个索引。|
key_len | 表示查询用到的索引长度（字节数），原则上长度越短越好 。 | 单列索引，那么需要将整个索引长度算进去； <br> 多列索引，不是所有列都能用到，需要计算查询中实际用到的列。<br> 注意：key_len只计算where条件中用到的索引长度，而排序和分组即便是用到了索引，也不会计算到key_len中。|
ref | - | - |
- | const | 当使用常量等值查询，显示const |
- | 字段名 | 当关联查询时，会显示相应关联表的关联字段 |
- | func | 如果查询条件使用了表达式、函数，或者条件列发生内部隐式转换，可能显示为func |
- | null | 其他情况 |
rows | 行数 | 以表的统计信息和索引使用情况，估算要找到我们所需的记录，需要读取的行数(越小越好) |
filtered | 百分比值 | 表示表里符合条件的记录数的百分比;在MySQL.5.7版本以前想要显示filtered需要使用explain extended命令。MySQL.5.7后，默认explain直接显示partitions和filtered的信息。 |
Extra | 不适合在其他列中显示的信息 | 一些额外描述 |
- | Using index | 我们在相应的 select 操作中使用了覆盖索引，通俗一点讲就是查询的列被索引覆盖，使用到覆盖索引查询速度会非常快，SQl优化中理想的状态 |
- | Using where | 查询时未找到可用的索引，进而通过where条件过滤获取所需数据，但要注意的是并不是所有带where语句的查询都会显示Using where |
- | Using temporary | 表示查询后结果需要使用临时表来存储，一般在排序或者分组查询时用到 |
- | Using filesort | 表示无法利用索引完成的排序操作，也就是ORDER BY的字段没有索引，通常这样的SQL都是需要优化的 |
- | Using join buffer | 在我们联表查询的时候，如果表的连接条件没有用到索引，需要有一个连接缓冲区来存储中间结果 |
- | Impossible where | 表示在我们用不太正确的where语句，导致没有符合条件的行。 |
- | No tables used | 我们的查询语句中没有FROM子句，或者有 FROM DUAL子句 | 

> 参考来源
- [不会看 Explain执行计划，劝你简历别写熟悉 SQL优化](https://juejin.im/post/5ec4e4a5e51d45786973b357)
- [MySQL官方文档-explain-output](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)
- [MySQL5.7中文文档](https://www.docs4dev.com/docs/zh/mysql/5.7/reference/explain-output.html#jointype_index_subquery)
- [一本彻底搞懂MySQL索引优化EXPLAIN百科全书](https://mp.weixin.qq.com/s/rem7Ds_QSnyhlrtNPByQcg)