---
title: MySQL在一个查询里做两次group by
date: 2020-03-19 00:34:38
updateDate: 2020-03-19 00:34:38
tags: MySQL
categories: MySQL
---
> [参考自](https://segmentfault.com/q/1010000019322585)

原始的语句:

```
SELECT email_id,count(1) as c FROM tbl_email_relation WHERE mail_id IN (xxx,xxx) AND customer_id != 0 and email_id = 3312172 GROUP BY email_id
```

<!--more-->

出来的数据是这样的

```
email_id | c |
111 | 4 |
```

实际上对应的原始数据是这样的:

```
email_id | mail_id |
111 | 111111 |
111 | 222222 |
111 | 222222 |
111 | 222222 |
```

而我们希望的数据是

```
email_id | c |
111 | 2 |
```

也就是说希望对mail_id进行二次排序

那么可以这样写

```
SELECT email_id,count(DISTINCT mail_id) FROM tbl_email_relation WHERE mail_id IN (xxx,xxx) AND customer_id != 0 and email_id = 3312172 GROUP BY email_id
```

出来的数据就是

```
email_id | c |
111 | 2 |
```

