---
title: Vim安装nerdtree
date: 2019-10-10 10:14:50
tags: Vim
categories: Vim
---
> 参考文章
> 1. [掘金-nerdtree安装](https://juejin.im/post/5c1fa6e4f265da6167205011)
> 2. [vim-github](https://github.com/scrooloose/nerdtree)

<!--more-->

##### 安装
```
1. 把项目download下来
$ git clone https://github.com/scrooloose/nerdtree.git ~/.vim/bundle/nerdtree

2. 在.vim下创建plugin和doc目录
$ mkdir -p ~/.vim/{plugin,doc}
 
3. 复制nerdtree的文件
$ cd nerdtree
$ cp plugin/NERD_tree.vim ~/.vim/plugin/
$ cp doc/NERD_tree.txt ~/.vim/doc/
```
安装好之后.
```
$ vim test.log
$ 在vim中输入:NERDTree
```
![vim nerdtree](https://raw.githubusercontent.com/Wuyiwai/Source/master/20191010101058.png)

##### 设置快捷键 
```
1. 修改vim配置
$ vim ~/.vimrc

2. 添加下列配置
map <F10> :NERDTreeMirror<CR>
map <F10> :NERDTreeToggle<CR>
```

##### 问题
1. cp之后如果遇到报错.把~/.vim/bundle/nerdtree目录下所有文件cp到~/.vim下即可,合并cp而不是替换cp