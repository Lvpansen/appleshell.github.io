---
title: git常用命令
date: 2019-11-26 20:13:15
tags: git
categories: git
---
git常用，但是有些命令记不住，用的时候就临时根据使用场景google，然后就又忘了。现在总结一波，方便以后查阅。
<!-- more -->

使用git命令之前，一定要掌握git中几个工作区的概念：

![git][img1]

> * Workspace：工作区
> * Index / Stage：暂存区
> * Repository：仓库区（或本地仓库）
> * Remote：远程仓库

[img1]:http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png

详细的常用命令请查看[阮一峰老师的文章][1]，有时间多看看，对于学习git很有帮助 。下面的内容是自己工作中遇到的一些场景，总结一下。

[1]:http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html

> ### 先建好了本地项目，之后需要推到远程仓库

    # 如果项目创建时没有初始化为git仓库，需要先执行
    $ git init

    # init后默认时master分支，如有必要，先创建并切换到自己的分支上
    $ git checkout -b myBranch

    # 本地仓库关联远程仓库
    $ git remote add origin git@github.com:appleshell/learn.git

    # 本地仓库代码推送到远程仓库
    $ git push origin <本地分支>:<远程分支>

> ### clone远程仓库用`git clone [url]`，clone下来的代码默认分支是master，有时候我们需要clone非master分支的代码

    $ git clone -b [branch] [url]

> ### 从远程仓库获取最新代码合并到本地分支

  方式一：

    # 拉取远程仓库指定分支代码并合并到本地分支
    $ git pull origin [branch]

  方式二：

    # 拉取远程仓库指定分支代码
    $ git fetch origin [branch]

    # 对比本地分支和远程分支差异
    $ git log -p [local_branch]..origin/[remote_branch]

    # 合并最新代码到本地分支
    $ git merge origin/[remote_branch]

  推荐方式二，可以在合并前查看差异，解决冲突；方式一直接合并，没法提前处理冲突。这里需要理解`git pull`和`git fetch`的[差异][2]

  [2]: https://www.zhihu.com/question/38305012

> ### 强制用远程仓库代码覆盖本地代码

    $ git fetch --all

    $ git reset --hard origin/master

    $ git pull

> ### 删除远程分支

    # 有必要可以先看一下远程有哪些分支
    $ git branch -r

    # 删除远程分支
    $ git push -d origin [remote_branch]

> ### git比较两个分支的文件的差异

    # 显示所有有差异的文件列表
    $ git diff [branch1] [barnch2] --stat

    # 显示指定文件的差异
    $ git diff [branch1] [barnch2] [文件路径]

    # 显示所有有差异的文件的详细差异内容
    $ git diff [branch1] [barnch2]

更多有关`git diff`命令相关的内容，点击[这里][3]

[3]:https://www.cnblogs.com/qianqiannian/p/6010219.html