---
title: mac下安装xdebug
date: 2019-09-18 11:45:26
tags: 
    - PHP 
    - Xdebug
categories: 
    - PHP 
    - Xdebug
---
> 参考文章
> 1. [Xdebug-install](https://xdebug.org/docs/install)

<!--more-->

##### 配置过程
```
1. pecl安装
$ pecl install xdebug
此步骤结尾处会输出:
# Build process completed successfully
# Installing '/usr/local/Cellar/php/7.3.9/pecl/20180731/xdebug.so'
# install ok: channel://pecl.php.net/xdebug-2.7.2
# Extension xdebug enabled in php.ini

2. 查看是否安装成功
$ php -m

如果输出
# [Zend Modules]
# Xdebug
# Zend OPcache
则安装成功
```

