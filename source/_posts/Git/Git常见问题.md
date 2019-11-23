---
title: Git常见问题
date: 2019-11-23 12:12:20
updateDate: 2019-11-23 12:12:20
tags: Git
categories: Git
---

###### Git status不能显示中文
1. 原因: 在默认设置下，中文文件名在工作区状态输出，中文名不能正确显示，而是显示为八进制的字符编码。
2. 解决办法 :将git 配置文件core.quotepath项设置为false。quotepath表示引用路径,加上--global表示全局配置
3. `git config --global core.quotepath false`