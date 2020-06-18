---
title: 了解Systemd
date: 2020-06-18 16:23:28
updateDate: 2020-06-18 16:23:28
tags: linux
categories: linux
---
> 参考文章:
> 1. [Systemd入门教程](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
> 2. [systemctl命令](https://man.linuxde.net/systemctl)
> 3. [journalctl工具基础介绍命令](https://man.linuxde.net/journalctl%e5%b7%a5%e5%85%b7%e5%9f%ba%e7%a1%80%e4%bb%8b%e7%bb%8d)

<!--more-->

##### 命令
以服务xxx.service为例

1. 启动服务 `systemctl start async-queue-notification.service`
2. 查看服务状态 `systemctl status async-queue-notification.service`
    ```
    $ systemctl status async-queue-notification.service
    
    ● async-queue-notification.service - async-queue-notification
       Loaded: loaded (/usr/lib/systemd/system/async-queue-notification.service; enabled; vendor preset: disabled)
       Active: active (running) since Thu 2020-06-18 10:04:30 CST; 4h 30min ago
      Process: 8868 ExecStop=/usr/bin/printf yes
     | /usr/local/php7/bin/php -c/usr/local/php7/etc/php-cli.ini vendor/bin/async-queue worker:stop --env=prod --channel=notification_prod (code=exited, status=0/SUCCESS)
     Main PID: 20500 (php)
        Tasks: 55
       Memory: 59.7M
       CGroup: /system.slice/async-queue-notification.service
               ├─20500 async-queue: master#notification_prod
               ├─20509 async-queue: monitor#notification_prod
               ├─20511 async-queue: consumer#notification_prod
               ├─20513 async-queue: consumer#notification_prod
               ├─22290 async-queue: consumer#notification_prod
               ├─22292 async-queue: consumer#notification_prod
               └─22294 async-queue: consumer#notification_prod
    
    Jun 18 10:04:30 crm-fpm-chrome-vpc systemd[1]: Started async-queue-notification.
    
    上面输出结果含义如下:
    ```

    1. Loaded行：配置文件的位置，是否设为开机启动
    2. Active行：表示正在运行
    3. Main PID行：主进程ID
    4. Status行：由应用本身（这里是 httpd ）提供的软件当前状态
    5. CGroup块：应用的所有子进程
    6. 日志块：应用的日志
3. 停止服务 `systemctl stop async-queue-notification.service`
4. 杀服务 `systemctl kill async-queue-notification.service`
5. 重启服务 `systemctl restart async-queue-notification.service`

##### 关于配置文件
1. 配置文件路径
    1. 一般在 `/usr/lib/systemd/system`
    2. 也有可能在`/etc/systemd/system`
2. 查看某个service的配置文件 `systemctl cat async-queue-notification.service`
    1. 输出
    ```
    [Unit]
    # 服务描述
    Description=async-queue-notification
    # 在网络初始化之后启动
    After=network.target
    
    [Service]
    # 服务类型
    Type=simple
    # 进程退出立即重启
    Restart=always
    
    # 工作目录
    WorkingDirectory=/xx/xx/xx
    # 启动命令
    ExecStart=/usr/local/php7/bin/php -c/usr/local/php7/etc/php-cli.ini vendor/bin/async-queue worker:listen --env=prod --channel
    # 停止命令
    ExecStop=/usr/bin/printf "yes\n" | /usr/local/php7/bin/php -c/usr/local/php7/etc/php-cli.ini vendor/bin/async-queue worker:st
    
    [Install]
    # 当系统以多用户方式启动时，这个服务需要被自动运行
    WantedBy=multi-user.target
    ```
    2. 解释
        1. Unit区块: 启动顺序与依赖关系
            1. Description: `当前服务的简单描述`
            2. After: `该服务需要在该字段代表的服务启动之后再启动`
            3. Before: `该服务需要在该字段代表的服务启动之前启动`
        2. Service区块: 该区块定义如何启动当前服务
            1. EnvironmentFile: `指定当前服务的环境参数文件。该文件内部的key=value键值对，可以用$key的形式，在当前配置文件中获取。`
            2. ExecStart: `启动进程时执行的命令。`
            3. ExecStop: `停止服务时执行的命令`
            4. ExecReload: `重启服务时执行的命令`
            5. ExecStartPre: `启动服务之前执行的命令`
            6. ExecStartPost: `启动服务之后执行的命令`
            7. ExecStopPost: `停止服务之后执行的命令`
            8. Type: 定义启动类型
                1. simple（默认值：`ExecStart字段启动的进程为主进程`
                2. forking：`ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程`
                3. oneshot：`类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务`
                4. dbus：`类似于simple，但会等待 D-Bus 信号后启动`
                5. notify：`类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务`
                6. idle：`类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合`
        3. Install区块: 
            1. WantedBy: `表示该服务所在的 Target`
                1. 解释: `Target`的含义是服务组，表示一组服务。`WantedBy=multi-user.target`指的是，该服务所在的 Target 是`multi-user.target`。而在`multi-user.target`这个组里的所有服务都将开机启动.这就是为什么`systemctl enable`命令能设置开机启动的原因
                2. 可用命令
                ```
                # 查看 multi-user.target 包含的所有服务
                $ systemctl list-dependencies multi-user.target
                
                # 切换到另一个 target
                # shutdown.target 就是关机状态
                $ sudo systemctl isolate shutdown.target
                ```
##### 修改配置文件后重启
    ```
    # 重新加载配置文件
    $ sudo systemctl daemon-reload
    
    # 重启相关服务
    $ sudo systemctl restart async-queue-notification.service
    ```

##### 关于日志
1. 查看systemd的日志
    ```
    # 查看该服务的日志
    $ journalctl -u async-queue-notification.service
    
    # 查看该服务当天的运行日志
    $ journalctl -u async-queue-notification.service --since today
    ```
2. 查看PHP层面的日志
    ```
    $ tail -f -n 1000 /data/php_errors.log
    ```