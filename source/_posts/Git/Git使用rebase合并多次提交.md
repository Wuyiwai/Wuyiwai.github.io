---
title: Git使用rebase合并多次提交
date: 2019-11-23 11:55:58
updateDate: 2019-11-23 11:55:58
tags: Git
categories: Git
---
> 参考文章
> 1. [掘金:使用rebase合并提交](https://juejin.im/post/5dce08ae5188254ea020ffaf)

<!--more-->

想要合并n次提交记录,有两种方法:
1. 从HEAD版本开始往后数n个版本
```
git rebase -i HEAD~n
```
2. 指定一个合并区间 ==startpoint== 和 ==endpoint==, 注意: **该区间指定的是一个前开后闭的区间,意思就是startpoint不参与合并**
```
git rebase -i [startpoint] [endpoint]
```
- -i 的意思是 ==--interactive==，即弹出交互式的界面让用户编辑完成合并操作
- **startpoint** 和 **endpoint** 指定了一个编辑区间
- 如果不指定**endpoint**，则该区间的终点**endpoint**默认是当前分支HEAD所指向的提交

##### 实践出真知
1. `git log`查看提交历史
```
前4条是这样: 
be7b37c (HEAD -> master) feat: 测试rebase-删除test.txt
05d4baf 测试rebase-1
639e94b 测试rebase
d2a1ecc (origin/master, origin/HEAD) test1-rebase
```
2. 合并过往三条提交记录, `git rebase -i HEAD~3`, 交互窗口如下:

![rebase交互弹窗](https://raw.githubusercontent.com/Wuyiwai/Source/master/rebase-1.png)

操作说明:
- pick：保留该commit（缩写:p）
- reword：保留该commit，但我需要修改该commit的注释（缩写:r）
- edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
- squash：将该commit和前一个commit合并（缩写:s）
- fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
- exec：执行shell命令（缩写:x）
- drop：我要丢弃该commit（缩写:d）

3. 按照需求,我们需要讲commit的内容编辑成如下:
```
pick 639e94b 测试rebase
s 05d4baf 测试rebase-1
s be7b37c feat: 测试rebase-删除test.txt
```
意思是: 把第二第三次的提交都合并到第一次提交上,保留第一次提交.
接下来`:wq`退出.此时git会压缩提交历史.如果这时候有冲突,需要修改,修改的时候要注意,保留最新的历史,不然修改就丢弃了.

4. 如果没有冲突,或者冲突已经解决,会出现如下的编辑窗口
![rebase-2](https://raw.githubusercontent.com/Wuyiwai/Source/master/rebase-2.png)
这是一个编辑commit信息的交互窗口.我们可以在这里编辑最后显示的commit信息
我们可以编辑成这样: 只保留了一句总结性的commit信息

```
  1 # This is a combination of 3 commits.
  2 # This is the 1st commit message:
  3
  4 feat: 测试rebase-删除test.txt
  5
  6 # Please enter the commit message for your changes. Lines starting
  7 # with '#' will be ignored, and an empty message aborts the commit.
  8 #
  9 # Date:      Sat Nov 23 11:18:26 2019 +0800
 10 #
 11 # interactive rebase in progress; onto d2a1ecc
 12 # Last commands done (3 commands done):
 13 #    squash 05d4baf 测试rebase-1
 14 #    squash be7b37c feat: 测试rebase-删除test.txt
 15 # No commands remaining.
 16 # You are currently rebasing branch 'master' on 'd2a1ecc'.
 17 #
 18 # Changes to be committed:
 19 #       deleted:    test.txt
 20 #
```

5. 接下来保存
```
$ git add .  # 将所有修改添加到暂存区
$ git rebase --continue # 继续rebase
```

6. 最后 `git log --oneline`查看历史
```
09553ad (HEAD -> master) feat: 测试rebase-删除test.txt
d2a1ecc (origin/master, origin/HEAD) test1-rebase
22fca5c feat: 更新13-169-191-229
```

7. 如果想要放弃这次压缩, 执行以下命令
```
git rebase --abort
```
