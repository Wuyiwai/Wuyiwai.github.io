---
title: PHP实现一些奇奇怪怪的需求
date: 2019-10-22 21:25:42
tags: PHP
categories: PHP
---
1. 字符串过滤除ASCII码之外的字符,使用场景:数据库字符集设置为ASCII,而数据来源有其他字符集的数据时
```
preg_replace('/[^(\x20-\x7F)]*/','', trim(strtolower($email)))
```