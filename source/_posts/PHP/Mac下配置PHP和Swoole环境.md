---
title: Mac下配置PHP和Swoole环境
date: 2019-09-13 03:40:05
tags:
  - PHP
  - Swoole 
categories:
  - PHP
  - Swoole 
---
> 记录一下配置Swoole环境的过程

前言: 本身Mac是自带了php的.通过`php -v`可以看到自带php版本是php7.1.略旧.直接上手php7.3.

<!--more-->

##### 准备工作,安装php和一些依赖
- brew install php(默认安装最新的).这个过程大部分时间取决于网速.
- brew的最后会执行`make install`.
- 如果`brew install xxx`成功,brew安装好的位置在`/usr/local/Cellar/`目录下;比如刚安装好的php 7.3的目录就为`/usr/local/Cellar/php/7.3.9/bin/php`
- 此时在cli环境中使用php -v显示的还是7.1.因为cli环境加载的配置是默认的php路径.所以要替换成刚安装的php7.3的版本.可以用`where php`查看php安装在哪里.如下
    ```
    $ where php
    /usr/local/Cellar/php/7.3.9/bin/php(这个可以)
    /usr/local/bin/php(这个也可以)
    /usr/bin/php
    ```
- 替换默认的php版本.
    - 假设用的是oh my zsh
    - vim ~/.zshrc
    - export PATH=/usr/local/Cellar/php/7.3.9/bin:$PATH
    - source ~/.zshrc
    - php -v检查是否生效
    - php.ini文件位于`/usr/local/etc/php/7.3/php.ini`.也可以用`php --ini`来定位.ini
    如下
    
    ```
    php --ini
    Configuration File (php.ini) Path: /usr/local/etc/php/7.3
    Loaded Configuration File:         /usr/local/etc/php/7.3/php.ini
    Scan for additional .ini files in: /usr/local/etc/php/7.3/conf.d
    Additional .ini files parsed:      /usr/local/etc/php/7.3/conf.d/ext-opcache.ini
    ```
    - 可以通过查看`php.ini`或者`phpinfo()`查看extension在/usr/local/lib/php/pecl/20180731
    
##### 编译swoole
> 参考资料:
> 1. [官方编译指南](https://wiki.swoole.com/wiki/page/6.html)

- [下载releases包](https://github.com/swoole/swoole-src/releases)
- 解压该releases包,重命名为swoole
- 接下来新手编译示例:
    ```
    $ cd swoole
    $ phpize 
    $ ./configure
    $ make 
    $ sudo make install
    ```
- 编译完成后,在`/usr/local/lib/php/pecl/20180731`.下会有swoole.so
- 最后,修改`php.ini`,加入`extension=swoole.so`
- 最后,`php -m`或者`phpinfo()`.查看是否成功加载swoole.so
- 这里做一下解释:phpize会检测php环境和生成对应configure文件.此时用的php版本和依赖取决于cli环境下制定的版本和对应的ini,conf等配置.所以此时用的是php7.3