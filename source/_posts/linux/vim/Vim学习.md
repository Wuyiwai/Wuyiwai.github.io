---
title: Vim学习
date: 2019-09-28 23:33:11
tags: Vim
categories: Vim
---
> 参考文章
> 1. [Vim练级手册](https://vim.wxnacy.com/#README)

<!--more-->

#### 键位操作
##### 快速移动光标
```
w           " 移动到下一个单词开头
e           " 移动到单词的结尾
b           " 移动到单词的开头
```

##### 上下左右
```
h  j  k l
← ↓ ↑ →
```

##### 行首行尾
```
0 快速跳到行首
$ 快速跳到行尾
```

###### 页头页尾
```
gg 快速跳到第一行
G 快速跳到最后一行
```

##### 查找替换
```
/word   " 输入 / 会进入 command 模式，再输入先要搜索的单词并回车进行搜索
?word   " / 是向光标以后搜索，? 是向前搜索
n       " 英文字母 n，根据 / 或 ? 搜索的方向定位到下一个匹配目标
N       " 与 n 相反，定位匹配目标
```

##### 替换
```
:n1,n2s/word1/word2/g   " n1, n2 表示数字，替换 n1 行到 n2 行的单词
:1,$s/word1/word2/g     " 全文替换，也可以写成 :%s/word1/word2/g
:1,$s/word1/word2/gc    " 全文替换，并出现确认提示
```

##### 复制粘贴
```
dd          " 删除当前行
ndd         " 向下删除 n 行
d1G / dgg   " 删除第一行到当前行的数据
dG          " 删除当前行到最后一行的数据
d$          " 删除当前字符到行尾
d0          " 从行首删除到当前字符
yy          " 复制当前行
nyy         " 从当前行开始复制 n 行
y1G / ygg   " 从第一行复制到当前行
yG          " 从当前行复制到最后一行
y0          " 从行首复制到当前字符
y$          " 从当前字符复制到行尾
p, P        " 黏贴，p 黏贴到光标下一行，P 黏贴到光标上一行
```

##### 撤销重做
```
u        " 撤销
ctrl+r   " 重做
```

#### 模式
##### insert模式
```
i   " 在光标前进入 insert 模式
I   " 在当前行左边第一个非空字符前进入 insert 模式，类似其他编辑器的 <c-a> 快捷键
a   " 在光标后进入 insert 模式
A   " 在当前行右边第一个非空字符前进入 insert 模式，类似其他编辑器的 <c-e> 快捷键
o   " 在光标的下一行插入
O   " 在光标的上一行插入
s   " 删除当前字符，并进入 insert 模式"
S   " 删除当前行，并进入插入模式
cc  " 删除当前行，并进入插入模式
```

#### vimrc配置文件
[参考配置](https://vim.wxnacy.com/#docs/vimrc)
```
set number "显示行号
set cursorline "突出显示当前行
set cursorcolumn "突出显示当前列
syntax on "开启语法高亮
filetype on "开启文件类型检测
filetype plugin on "开启插件支持
filetype indent on "开启文件类型相应的缩进规则
set showmode  "显示当前vim模式
set showmatch "显示匹配的括号
set hlsearch "高亮搜索项
set tabstop=4 "键入tab的步长
set laststatus=2   " 底部显示两行状态栏"
```

##### 高级键位
```
ZZ 直接退出
:q! 强制退出
:w! 强制保存
:w ! sudo tee % 在vim中强制使用sudo保存以解决权限不够的问题
:w newfilename 另存为
```

#### 多窗口操作
##### 分割窗口
```
:sp filename    " 上下分割窗口
:vsp filename   " 左右分割窗口
```

##### 移动光标
```
<c-w>h[j[k[l]]]     " 根据方向键移动光标到该方向的窗口上
```

##### 调换窗口位置
```
<c-w>x      " 对调左右或上下两个对应的窗口
```

##### 退出窗口
```
<c-w>q
一般也可以先移动光标到该窗口,然后:q
```