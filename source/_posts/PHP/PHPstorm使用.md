---
title: PhpStorm使用
date: 2019-10-24 01:30:24
tags: PHP
categories: PHP
---
> 参考文章
> 1. [PHPstorm Tips](https://phpstorm.tips/tips/)
> 2. [Be Awesome in PHPStorm](https://www.bilibili.com/video/av40212757?from=search&seid=9863044042153804342)

记录PhpStorm使用技巧和使用中遇到的问题

<!--more-->

##### 日常使用
1. 主题
    1. [Material theme ui](http://plugins.jetbrains.com/plugin/8006-material-theme-ui)
2. 快速操作
    1. F2 快速跳到error处

##### 使用遇到的问题
1. PHPstorm升级后无法保存sftp的配置,提示`user name is specified`
    1. 原因:phpstorm默认使用mac的keychain来处理密码,可能在升级过程中删除了keychain的key
    2. 解决办法：PHPstorm preferences > Passwords > In Keepass