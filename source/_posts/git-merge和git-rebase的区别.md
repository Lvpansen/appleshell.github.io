---
title: git merge和git rebase的区别
date: 2019-11-29 18:22:47
tags: git
categories: git & github
---
`git merge`和`git rebase`所做的事是一样的，都是将一个分支上的代码合并到另一个分支上，但是两个合并后所产生的提交记录会有所不同，而且在处理代码冲突上也有所不同。
<!-- more -->

> ## git merge

* master分支的最新提交记录是**commit-master1**
* 从master分支上切出feature分支
* 在feature分支上进行了开发，产生了两个提交记录：**commit-feature1**和**commit-feature2**

情况1：master分支上没有新的提交记录

* 当前分支为master，执行 `git merge feature`，**commit-feature1**和**commit-feature2**两个提交记录会被合并到master分支上，这两个提交记录位于**commit-master1**前面，feature分支没有变化

情况2：master分支上有新的提交记录**commit-master2**（相当于你在feature分支上开发时，有人往master分支上提交了代码）

* 当前分支为master，执行 `git merge feature`，**commit-feature1**和**commit-feature2**两个提交记录会被合并到master分支上，这两个提交记录会和**commit-master2**提交记录按照时间顺序排在**commit-master1**前面。最后生成一个新的commit提交记录**commit-master-merge**（这个记录里可能会包含解决的冲突），位于提交记录的最前面。feature分支也是没有变化

可以实践一下，然后查看git的图形化的提交记录

[参考一][1]

[1]: https://www.html.cn/archives/10077