---
title: Git本地多账号工作.md
date: 2019-09-03 17:12:31
tags: Git
categories: Git
---
> 1. 最近遇到这样一种情况，在一台电脑上需要设置两个git账号。两个账号都有其作用。所以记录一下设置多账号的过程
> 2. 参考教程：https://blog.csdn.net/u014166319/article/details/78570868

<!--more-->

### 1. 分别生成新的公钥和私钥

```
ssh-keygen -t rsa -C "your_email"
```

1. 如果是单一账号的情况下，默认在.ssh目录下生成id_rsa和id_rsa.pub，但是重复操作会覆盖结果，所以谨记需要针对不同的账号设置不同的文件名，比如id_rsa_jack和id_rsa_tim
2. mac下一般放在~/.ssh/下

### 2. 配置ssh agent
1. ssh-agent类似于一个本地的密钥管理器。默认读取的是默认生成的id_rsa，所以要再添加另外一个账号的私钥/
```
ssh-agent bash
ssh-add ~/.ssh/id_rsa
```

### 3. 接下来添加公钥（ssh keys）到git账号的ssh配置中

### 4. 配置~/.ssh/config文件

```
//如果不存在，则创建config文件
touch ~/.ssh/config
```
1. 配置config信息
```
#该文件用于配置私钥对应的服务器
#account1(email)
 Host git@github.com
 HostName https://github.com
 User jack //这里填写username
 IdentityFile ~/.ssh/id_rsa_jack
######################################
#account2(email)
 Host git@gitlab.com
 HostName https://gitlab.com
 User tim//这里填写username
 IdentityFile ~/.ssh/id_rsa_tim
```

### 5. 测试是否可以连接git
```
ssh -T git@github.com

//if success will return
//Hi BeginMan! You've successfully authenticated, but GitHub does not provide shell access.
//else return 
//some errors
```

### 6. 最后，如何切换账号。
1. 首先，一般commit的时候，我们会在这之前已经做一步设置

```
git config --global user.name "your_name"
git config --global user.email  "your_email"
```
2. 然后，总不能每次切换都手动换一次，所以可以提供两个思路
    1. 可以写个两个shell，一个为changeToTim.sh，一个为changeToJack.sh
        1. 创建shell
        ```
        //changeToTim.sh
        git config --global user.name "your_name"
        git config --global user.email  "your_email"
        ```
        2. 执行shell，切换账户
        ```
        $ ./changeToTim.sh
        ```
    2. sourceTree
        1. sourceTree本身支持bitbucket账户
        2. 需要切换账户时，在commit时，填写commit message的框框左上角有个头像，点击可以设置另外的账号，填写一次，这个库的设置就保存好了。
        3. 这个方法比shell来的方便。
       
#### 可能遇到的问题 
1. ssh -T xx@xx.com 提示 git@gitee.com: Permission denied (publickey).解决如下：
    1. ssh-add xx/id_rsa
    2. ssh -T git@gitee.com





