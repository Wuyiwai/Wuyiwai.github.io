---
title: brew update无响应
date: 2019-10-10 11:05:37
tags: others
categories: others
---
> 参考文章
> 1. [brew update无响应](https://xu3352.github.io/mac/2018/09/06/mac-homebrew-update-slowly)

<!--more-->

##### 更换brew镜像源
```
# 进入brew主目录
$ cd `brew --repo`

# 更换镜像
$ git remote set-url origin https://git.coding.net/homebrew/homebrew.git

# 测试效果
$ brew update
```

##### 几个镜像:
- https://git.coding.net/homebrew/homebrew.git - Coding
- https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git - 清华
- https://mirrors.ustc.edu.cn/brew.git - 中科大


##### 切换回官方源：
```
# 重置brew.git
$ cd "$(brew --repo)"
$ git remote set-url origin https://github.com/Homebrew/brew.git

# 重置homebrew-core：
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```
