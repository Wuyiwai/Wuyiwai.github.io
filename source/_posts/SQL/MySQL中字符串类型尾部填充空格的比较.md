---
title: MySQL中字符串类型尾部填充空格的比较
date: 2020-08-05 10:50:20
updateDate: 2020-08-05 10:50:20
tags: MySQL
categories: MySQL
---

> 文档
> 1. [比较中的尾随空间处理](https://dev.mysql.com/doc/refman/8.0/en/charset-binary-collations.html#charset-binary-collations-trailing-space-comparisons)
> 2. [CHAR和VARCHAR类型](https://dev.mysql.com/doc/refman/8.0/en/char.html)

<!--more-->

背景:
业务中有这么两行sql:
1. `SELECT subject, count(*) as total FROM test WHERE task_id = 111 and isdelivered = 1 GROUP BY subject;`
2. `SELECT subject, count(*) as total FROM test WHERE task_id = 111 and status! = 6 GROUP BY subject;`

1. 第一个查询结果为:
```
subject | total
'xxx' | 3590
'wireless car ' | 7362
```

2. 第二个查询结果为: 
```
subject | total
'xxx' | 5757
'wireless car  ' | 11683
```

注意: 这里的查询结果中的`'wireless car  '`中的`''`只是为了区分分界线,其真实内容为`wireless car` + `  `(两个空格).后续查询结果为了突出显示空格,也作此处理.

一眼就可以看出,两个查询得出的`group by`并不同.这是因为在进行`group by`比较时,`group by`的字段`subject`为`varchar`类型.

通过查询可以得知`utf8bm4_gener_ci`排序规则的`PAD_ATTRIBUTE`为`PAD SPACE`
```
mysql> SELECT COLLATION_NAME, PAD_ATTRIBUTE FROM INFORMATION_SCHEMA.COLLATIONS WHERE COLLATION_NAME LIKE 'utf8mb4%';
+----------------------------+---------------+
| COLLATION_NAME             | PAD_ATTRIBUTE |
+----------------------------+---------------+
| utf8mb4_general_ci         | PAD SPACE     |
```
说明在对该排序规则的比较中,不比较尾部空格.
我们可以看一个例子:
```
mysql> SET NAMES utf8mb4 COLLATE utf8mb4_bin;
mysql> SELECT 'a ' = 'a';
+------------+
| 'a ' = 'a' |
+------------+
| 1 |
+------------+
mysql> SET NAMES utf8mb4 COLLATE utf8mb4_0900_bin;
mysql> SELECT 'a ' = 'a';
+------------+
| 'a ' = 'a' |
+------------+
| 0 |
+------------+
```
结论:
对于排序规则为utf8mb4_general_ci的列,pad_attribute为pad space.

对于非二进制字符串（CHAR， VARCHAR和TEXT 值），字符串排序规则属性确定比较字符串末尾空格的方式：
对于`PAD SPACE`排序规则，比较中的尾部空格不重要；比较字符串时不考虑尾随空格。
`NO PAD `归类将尾随空格视为比较中的重要内容，就像其他任何字符一样。

对于二进制字符串（BINARY， VARBINARY和BLOB ），所有字节在比较中都是有效的，包括尾随空格：

所以可以用`binary`进行显示转换进行`group by`比较.

所以可以对`sql`进行如下修正:
```
SELECT subject, count(*) as total FROM test WHERE task_id = 111 and isdelivered = 1 GROUP BY binary subject;
+--------------------------+-------+
| subject                  | total |
+--------------------------+-------+
|xxx charger	           | 3590 |
|'wireless  car  charger ' | 3673 |
|'wireless  car  charger  '| 3673 |
+--------------------------+-------+
```
即可得到正确的比较结果.