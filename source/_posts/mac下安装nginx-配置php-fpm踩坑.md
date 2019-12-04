---
title: 'mac下安装nginx,配置php-fpm踩坑'
date: 2019-12-04 10:36:34
updateDate: 2019-12-04 10:36:34
tags: ENV
categories: ENV
---
> 参考文章
> 1. [brew安装nginx](https://www.jianshu.com/p/026d67cc6cb1)
> 2. [解决mac下自带php-fpm和brew安装的php-fpm的冲突](https://blog.csdn.net/darthas/article/details/53980644)

<!--more-->

##### 安装nginx
通过`brew`安装nginx
```
brew install nginx
brew services start nginx
访问: http://localhost:8080
```

nginx常用命令
```
nginx  #启动nginx
nginx -s quit  #快速停止nginx
nginx -V #查看版本，以及配置文件地址
nginx -v #查看版本
nginx -s reload|reopen|stop|quit   #重新加载配置|重启|快速停止|安全关闭nginx
nginx -h #帮助
```

1. nginx文件目录
    1. nginx安装文件目录
        1. /usr/local/Cellar/nginx
    2. nginx配置文件目录
        1. /usr/local/etc/nginx
    3. config文件目录
        1. /usr/local/etc/nginx/nginx.conf

##### 配置php-fpm
1. mac下默认自带了php和php-fpm,那么在使用的时候会产生一些冲突
    1. 系统自带的php-fpm在/usr/sbin目录下，
    2. 通过brew安装的php中php-fpm在/usr/local/sbin目录，
    3. 默认启动PHP-fpm时系统会找到/usr/sbin目录的文件，他的默认php-fpm.conf为/private/etc目录，默认上不存在的,所以会报错导致无法启动

解决办法:
```
把/usr/local/sbin目录加入环境变量最前面，覆盖系统的自带设置
vim ~/.bashrc
输入export PATH=/usr/local/sbin:/usr/local/bin:$PATH
source ~/.bashrc
```