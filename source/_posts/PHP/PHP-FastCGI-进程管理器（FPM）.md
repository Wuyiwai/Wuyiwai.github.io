---
title: PHP FastCGI 进程管理器（FPM）
date: 2019-09-04 22:56:11
tags: PHP
categories: PHP
---
> 参考文章
> 1. [PHP-manual](https://www.php.net/manual/zh/install.fpm.php)
> 2. [百度百科-fastcgi](https://baike.baidu.com/item/fastcgi)
> 3. [LAMP架构下的模型分析](https://cloud.tencent.com/developer/news/326669)

#####  CGI是什么

CGI全称是“通用网关接口”(Common Gateway Interface)，web服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序一般运行在网络服务器上。 CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。

##### FastCGI是什么
1. FastCGI全称**快速通用网关接口**(FastCommonGatewayInterface).
2. FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。
3. 它还支持分布式的运算, 即FastCGI程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。
4. FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等

##### PHP-FPM(PHP FastCGI Process Manager)是什么
1. PHP-FPM是一个PHP FastCGI的进程管理器,管理PHP进程池.主要可以有效地控制内存和进程,可以平滑重载PHP配置
2. PHP-FPM是因为PHP-FastCgi的出现,可以管理php-fastcgi
3. 简单点来说就是PHP-FPM管理启动一个masger进程和多个worker进程的程序.
4. PHP-FPM是多进程模式,每个子进程是单一线程,通过master进程管理worker进程.启动进程的方式可以分为动态模式和静态模式.
    1. 动态模式(dynamic)
        1. 一开始开启一定数量的PHP-FPM进程,请求量变多,动态增加进程数;空闲时释放.
    2. 静态模式(static)
        1. 子进程的数量是固定的(**配置项:pm.max_children**)
    3. 按需分配(ondemand)
    4. 配置项:
        1. **pm.max_children**----pm设置为 static 时表示创建的子进程的数量，pm 设置为 dynamic 时表示最大可创建的子进程的数量。必须设置。
        2. **pm.start_servers**----设置启动时创建的子进程数目。仅在 pm 设置为 dynamic 时使用。默认值：min_spare_servers + (max_spare_servers - min_spare_servers) / 2。
        3. **pm.min_spare_servers**----设置空闲服务进程的最低数目。仅在 pm 设置为 dynamic 时使用。必须设置。
        4. **pm.max_spare_servers**----设置空闲服务进程的最大数目。仅在 pm 设置为 dynamic 时使用。必须设置。
    2. 如何选择
        1. 看业务.
            1. 看业务的请求是什么类型的.全天平稳,可以预测的峰值?还是峰值不定类似于微博爆题这种.如果是全天稳定,或者说峰值可以预测的.看机器静态模式固定进程数能否轻松抗住,如果可以,进程数适当调整一下,保证抗压的2-3倍即可选用静态模式.如果是峰值不定的且分分钟可能超出预期的,可以考虑选用动态,并且结合DBA扩容.
        2. 看配置.高配选静态,低配选动态.
5. 优缺点分析:
    1. 进程模型是多进程同步阻塞.一个进程处理一个链接.最大的优势就是稳定可靠,编程简单.
    2. 明显的缺点是:处理大规模并发的长连接的时候吃力.阻塞容易成为痛点.