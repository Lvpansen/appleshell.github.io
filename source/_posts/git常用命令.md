---
title: git常用命令
date: 2019-11-26 20:13:15
tags: git
categories: git
---
git常用，但是有些命令记不住，用的时候就临时根据使用场景google，然后就又忘了。现在总结一波，方便以后查阅。
<!-- more -->

1. 先建好了本地项目，之后需要推到远程仓库
****

    # 如果项目创建时没有初始化为git仓库，需要先执行
    $ git init

    # init后默认时master分支，如有必要，先创建并切换到自己的分支上
    $ git checkout -b myBranch

    # 本地仓库关联远程仓库
    $ git remote add origin git@github.com:appleshell/learn.git

    # 本地仓库代码推送到远程仓库
    $ git push origin <本地分支>:<远程分支>

    