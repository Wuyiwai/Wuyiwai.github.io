---
title: 从 PHP 7.2.x 移植到 PHP 7.3.x的变更
date: 2019-09-18 11:01:06
tags: PHP
categories: PHP
---
> 参考文章
> 1. [从 PHP 7.2.x 移植到 PHP 7.3.x](https://www.php.net/manual/zh/migration73.php)
### New Features
##### PHP Core

### 新函数
- [array_key_first ( array $array ) : mixed](https://www.php.net/manual/zh/function.array-key-first.php) Get the first key of the given array **without affecting the internal array pointer**.
- [array_key_last ( array $array ) : mixed](https://www.php.net/manual/zh/function.array-key-last.php) Get the last key of the given array **without affecting the internal array pointer**.

### 废弃的功能
##### 大小写不敏感的常量
大小写不敏感的常量声明现已被废弃。将 TRUE 作为第三个参数传递给 define() 将会导致一个废弃警告。大小写不敏感的使用（在读取时使用一个与声明时不同的大小写方式）也已被废弃。

##### 在字符串中搜索非字符串内容
废弃：将一个非字符串内容传递给字符串搜索函数。 在将来所有待搜索的内容都将被视为字符串，而不是 ASCII 编码值。如果需要依赖这个特性，你应该 要么显示地进行类型转换（转为字符串），或者显示地调用 chr()。 以下是受到影响的方法：
- strpos()
- strrpos()
- stripos()
- strripos()
- strstr()
- strchr()
- strrchr()
- stristr()
