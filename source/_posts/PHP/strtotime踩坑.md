---
title: strtotime踩坑
date: 2019-11-25 23:25:19
updateDate: 2019-11-25 23:25:19
tags: PHP
categories: PHP
---
> 参考文章:
> 1. [风雪之隅-令人困惑的strtotime](http://www.laruence.com/2018/07/31/3207.html)

###### +1 month / -1 month踩坑

```
<?php
echo "[2018-07-31经过(-1 month)之后是:]----" , date("Y-m-d",strtotime("-1 month", strtotime('2018-07-31'))) . PHP_EOL;
//输出: 2018-07-01
```

<!--more-->

此时逻辑是:
1. 2018-7-31 减去一个月 变成 2018-6-31
2. 但是6月没有31天,6月最长是30天
3. 所以顺延一天,6.31 = 7.1

我们可以做个实验证明上面的逻辑.
```
echo "[2017-06-31经过strtotime之后是:]----" , date("Y-m-d", strtotime("2017-06-31")) . PHP_EOL;
//输出: 2018-07-01
```

同理,只要遇到月尾的时候+1 或者 -1 month,不是实际存在的日期都会发生进退,如下:
```
var_dump(date("Y-m-d", strtotime("-1 month", strtotime("2017-03-31"))));
//输出2017-03-03
var_dump(date("Y-m-d", strtotime("+1 month", strtotime("2017-08-31"))));
//输出2017-10-01
var_dump(date("Y-m-d", strtotime("next month", strtotime("2017-01-31"))));
//输出2017-03-03
var_dump(date("Y-m-d", strtotime("last month", strtotime("2017-03-31"))));
//输出2017-03-03
```

那么该如何避免: PHP 5.3以后可以使用 "==first day of==" 和 "==last day of=="等修饰词
```
var_dump(date("Y-m-d", strtotime("last day of -1 month", strtotime("2017-03-31"))));
//输出2017-02-28
var_dump(date("Y-m-d", strtotime("first day of +1 month", strtotime("2017-08-31"))));
////输出2017-09-01
var_dump(date("Y-m-d", strtotime("first day of next month", strtotime("2017-01-31"))));
////输出2017-02-01
var_dump(date("Y-m-d", strtotime("last day of last month", strtotime("2017-03-31"))));
////输出2017-02-28
```
