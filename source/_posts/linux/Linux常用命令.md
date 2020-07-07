---
title: Linux常用命令
date: 2020-07-07 14:34:26
updateDate: 2020-07-07 14:34:26
tags: linux
categories: linux
---
> 参考文章:
> 1. [Linux常用命令 & 实用命令万字总结 作者: Java小咖秀](https://juejin.im/post/5ee444ea6fb9a047ca10f796)

#### ls

1.  `ls -l` 展示额外信息
2.  `ls -lS` 按文件大小降序显示
3.  `ls -lt` 按修改时间倒序显示
4.  `ls -a` 查询所有文件(包括隐藏文件);linux下'.'开头的文件默认隐藏
5.  `ls -F` 以'/'结尾的方式展示文件夹
6.  `ls -r` 按字典序倒序展示
7.  `ls -R` 递归展示子目录
8.  `ls -g` 不输出所有者信息
9.  `ls -lG` 隐藏所有者组信息
10.  `ls -li` 显示文件的索引号

<!--more-->

#### cat

1.  `cat -n filename` 查看并显示行号
2.  `cat --help` 显示帮助信息


#### 进程管理基础

##### 进程的属性

1.  进程ID : (pid) 唯一的数字标示，区分不同的进程。
2.  进程有父进程和子进程。
3.  启动进程的用户ID(uid)和用户的属组。
4.  进程的三种状态:
    1.  运行 -- R
    2.  休眠 -- S
    3.  僵尸 -- Z
5.  进程的优先级: 取值范围(-20,19) ,数值越小优先级越大,默认为0。
6.  进程链接的终端。
7.  进程占用资源情况。

##### 进程管理工具

```
ps:   查看进程。
top:  可以查看进程的动态信息。
kill:  杀进程。
pstree:  查看进程树。
pgrep:  搜进程。
lsof:  查看进程打开的文件
```

#### 进程管理实战

##### ps

1.  常用参数

    1.  `a`  显示所有用户的进程
    2.  `r`  显示运行中的进程               
    3.  `l` 长格式输出               
    4.  `u` 按用户名和启动时间的顺序来显示进程               
    5.  `f` 用树形格式来显示进程               
    6.  `x` 显示没有控制终端的进程

2.  BSD格式显示进程

    ```
    $ ps aux
    
    属性详解
    USER: 进程的属主
    PID: 进程的ID
    %CPU: 进程占cpu百分比
    %MEM: 进程占内存的百分比
    VSZ: 进程占用虚拟内存大小
    RSS: 固定内存使用数量
    STAT 进程状态
      R    正在运行可中在队列中可过行的；
      S    处于休眠状态；
      T    停止或被追踪；
      Z    僵尸进程；
      N    优先级较低的进程
      L    有些页被锁进内存；
      s    进程的领导者（在它之下有子进程）
    START 启动进程的时间；
    TIME 进程消耗CPU的时间；
    COMMAND 命令的名称和参数;
    
    1. 按照cpu从小到大排序
    	$ ps aux --sort %cpu
    2. 按照cpu从大到小排序
    	$ ps aux --sort -%cpu
    ```

3.  标准格式显示进程unix风格

    ```
    $ ps -ef
    ```

##### top 动态显示进程信息

```
~ top

top - 11:21:51 up 5 days, 22:11,  8 users,  load average: 1.05, 1.22, 1.29
Tasks: 436 total,   1 running, 433 sleeping,   0 stopped,   2 zombie
Cpu(s):  6.0%us, 10.7%sy,  0.0%ni, 82.9%id,  0.1%wa,  0.0%hi,  0.2%si,  0.1%st
Mem:  16269248k total, 11458624k used,  4810624k free,   670024k buffers
Swap:  4112380k total,    12388k used,  4099992k free,  4575232k cached

 PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
1612 root      20   0  493m  92m 5228 S  4.0  0.6 377:19.05 mongod
29023 nobody    20   0 1481m 103m  93m S  1.7  0.7   1:30.81 php-fpm
29024 nobody    20   0 1482m  99m  88m S  1.0  0.6   1:28.46 php-fpm
 1497 elastics  20   0 6814m 2.4g  21m S  0.7 15.7  71:53.93 java
 1667 root      20   0  666m  73m  42m S  0.7  0.5  43:50.86 mongod
24190 root      20   0 15300 1696 1004 R  0.7  0.0   0:00.42 top
  989 root      16  -4 29772  740  620 S  0.3  0.0  13:10.55 auditd
  
前五行为统计信息
第一行: top          
11:21:51         当前时间
up 5 days,22:11  系统运行时间
8 users          当前登录用户数量
load average: 1.05, 1.22, 1.29 系统负载,任务队列平均长度 1分钟,5分钟,15分钟前到现在的平均值。
  
第二三行：
Tasks:
  436 total     进程总数
  1 running     正在运行的进程数
  433 sleeping  睡眠进程数
  0 stopped     停止进程数
  2 zombie      僵尸进程
Cpu:
  6.0%us        系统用户进程占用cpu百分比
  10.7%sy       内核进程占用cpu百分比
  0.0%ni        用户进程空间内改变过优先级的进程占用cpu百分比
  82.9%id       空闲cpu百分比
  0.1%wa        等待输入输出的cpu时间百分比
  0.0%hi        硬件cpu中断占用百分比
  0.2%si        软中断占用百分比
  0.1%st        虚拟机占用百分比
  
第四五行:
Mem:  16269248k total, 11458624k used,  4810624k free,   670024k buffers
Swap:  4112380k total,    12388k used,  4099992k free,  4575232k cached
Mem:
	16269248k total  物理内存总量
	11458624k used   空闲内存总量
	4810624k free    使用的物理内存总量
	670024k buffers  内核缓存的内存量
Swap:
	4112380k total   交换区总量
	12388k used      空闲交换区总量
	4099992k free    使用的交换区总量
	4575232k cached  可利用的内存量
	
标题行: PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
  PID:  进程id
  USER: 进程所有者用户名
  PR:   优先级
  NI:   进程优先级，nice值，负值 -> 高优先级，正值 -> 低优先级
  VIRT: 虚拟内存总量 virt=swap + res
  RES:  实际使用内存大小
  SHR:  共享内存大小
  S:    进程状态
     d: 不可中断的睡眠状态
     r: 运行
     s: 睡眠
     t: 跟踪
     z: 僵尸进程
  复制代码
  %CPU: 上次更新到现在cpu时间占用百分比
  %MEM: 进程使用物理内存百分比
  TIME+: 进程使用cpu的时间总计，单位 1/100秒
  COMMAND: 命令行
```

##### kill

语法:

```
kill [-s signal|-p] [-q sigval] [-a] [--] pid...
kill -l [signal]
```

常用参数:

```
-l 列出所有信号名称  
-s 指定发送信号（默认）
-u 指定用户
```

常用命令:

```
1. 列出所有信号名称
	$ kill -l
2. 强行终止进程
	$ kill -9 pid
3. 终止进程
	$ kill -15 pid
4. 杀死指定用户的所有进程
	$ kill -u superh
```



#### 文件查找

```
which 查看执行文件的位置
whereis 查看可执行文件位置和相关文件
locate 配合数据库缓存,快速查看文件的位置
grep 过滤匹配,文件搜索
find 根据条件查看文件
```

##### which

1.  `which findtarget` 一般不加参数使用

##### whereis

1.  `whereis findtarget` 一般不加参数使用

##### locate

1.  `locate findtarget` 跟find差不多,搜索效率更高.当天新建的文件需要手动updatedb

##### grep

1.  `grep 参数 目标值 文件`
2.  参数:
    1.  `-v`  取反(查看没有查询目标的行)
    2.  `-i`  忽略大小写
    3.  `-n`  输出的时候同时打印行号
    4.  `^*` 以*开头
    5.  `*$` 以*结尾
    6.  `^$` 空行

##### find

1.  `find 路径 参数 输出`
2.  常用参数
    1.  路径: 用`.`表示当前目录, 用`/`表示系统根目录
    2.  `-print`: 显示的时候`\n`做定界符,换行
    3.  `-print0`: 与`xargs`配套使用,以`\0`作为定界符
3.  常用命令选项:
    1.  `-name` 按文件名查找文件
    2.  `-perm` 按文件权限查找文件 666 777 等 
    3.  `-depth` 在查找文件时,先查找当前目录,再查找子目录
    4.  `-user` 按文件属主查找文件
    5.  `-atime`,`-ctime` 单位是天
    6.  `-mmin`,`-cmin`,`-amin`单位是分钟
    7.  `-size n [c]` 查找文件长度为n块的文件，带有c时表示文件长度以字节计
    8.  `-follow` 如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。
4.  应用:
    1.  查看当前目录下`test.`开头的文件 `find . -name 'test*'`
    2.  查看当前目录下test.开头的文件,输出结果换行显示(默认) `find . -name "test*" -print`
    3.  查看当前目录下test.开头的文件,输出结果不换行显示 `~ find . -name "test*" -print0`



#### 定时任务

1.  `crontab 参数`

2.  常用参数

    ```
    crontab -u   #指定用户的cron信息
    crontab -l   #列出当前用户下的cron服务的信息
    crontab -u   user -l   #列出指定用户的cron服务的信息
    crontab -r   #删除cron服务
    crontab -e   #编辑cron服务
    crontab -r -u user   #删除指定用户的定时任务。
    ```

3.  编辑cron服务 `crontab -e`
4.  查看当前用户下任务内容 `crontab -l`
5.  删除任务 `crontab -r` `,` `crontab -l`
6.  查看qa这个用户下的任务内容 `crontab -u qa -l`
7.  删除qa这个用户下的定时任务 `rontab -u qa -r` , ` crontab -u qa -l`



#### 系统任务

系统的任务调度配置文件在`/etc/crontab`下

1.  定时任务格式

2.  `*分钟(0-59)`    ` *小时(0-23) `    `*日期(1-31)`    ` *月份(1-12)`    ` *星期(0-7)`    ` command(执行的命令)`

3.  时间的一些特殊符号:

    1.  `*` 代表任意数字
    2.  `/` 时间的间隔频率
    3.  `-` 从某几个数字到某几个数字
    4.  `,` 几个指定的数字

4.  例子:

    ```
    #每晚的20:30重加载nginx
    30 20 * * * /usr/local/nginx/sbin/nginx -s reload 
    
    #每月1、10、15日的5:30重加载nginx
    30 5 1,10,15 * * /usr/local/nginx/sbin/nginx  -s reload  
    
    #每天20: 00至22 : 00之间每隔30分钟重加载nginx
    0,30 20-22 * * * /usr/local/nginx/sbin/nginx -s reload 
    
    每星期六的10 : 00 pm 重加载nginx
    0 22* * 6 /usr/local/nginx/sbin/nginx  -s reload 
    
    #每一小时重加载一次nginx
    0 */1 * * * /usr/local/nginx/sbin/nginx -s reload 
    ```



#### 权限管理

##### 角色

```
超级用户 root UID: 0 最高的管理权限
系统用户 UID:1-999(rhel7  centos7) 1-499(rhel6)
本地用户 UID:1000+(rhel7 centos7)  500+(rhel6)
```

##### linux组分类

```
超级用户组： root GID:0
普通用户组 ： 
  系统用户组: GID 1---999
  本地用户组: GID 1000+
```

##### linux用户和组的关系

一个用户属于一个组，也可以属于多个组，多个用户可以在一个组，可以在多个组。

##### linux用户和组的相关配置文件

```
1./etc/passwd: 用户 and 属性信息。
2./etc/group: 组 and 属性信息。
3./etc/shadow: 用户密码 and 属性信息。
4./etc/gshadow: 组密码 and 属性信息。
5./etc/default/useradd: 设置添加用户规则文件。
6./etc/login.defs: 设置用户账号限制。
```

##### linux用户信息文件/etc/passwd

```
$ cat /etc/passwd

root:x:0:0:root:/root:/bin/zsh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

举例:
root:x:0:0:root:/root:/bin/zsh
1.登陆用户名:root
2.用户密码占位符:x
3.用户UID:0
4.用户组GID:0
5.对用户账户的描述:root
6.用户家目录位置:/root
7.用户默认的shell:/bin/zsh (默认是/bin/bash)
```

#####  linux密码信息文件

该文件只有root用户有读权限，每一行存一个用户的记录，每个属性用冒号分割。

```
$ cat /etc/shadow

root:xxx
ZTY8r00/.:17769:0:99999:7:::
bin:*:17110:0:99999:7:::

举例:
bin:*:17110:0:99999:7:::
1.登录用户名: bin
2.加密的密码:  * 号代表密码被锁定
3.最近更改密码的日期: 17110代表到1970-1-1密码不可改的天数
4.密码修改期限： 99999 代表永远不用改,如果是其他数字则表示从1970-1-1内的多少天必须修改密码。
5.更改密码最大有效天数:  代表密码保持有效的最大天数。
6.密码过期警告:密码到正式失效前有多少天(-1,永远不提示)
7.密码过期后多少天禁用用户:可登陆，单不能操作。
8.用户被禁用日期:多少天后账号过期，不能登陆。
9.保留参数
```

##### linux组信息文件

用户组的组信息存放在这，一行存一个组记录，属性用冒号分割。

```
$ cat /etc/group

root:x:0:
bin:x:1:

举例: root:x:0:
1.组名：root
2.组密码占位符: x
3.组GID: 0
```