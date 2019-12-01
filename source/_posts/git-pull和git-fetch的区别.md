---
title: git pull和git fetch的区别
date: 2019-11-28 00:17:25
tags: git
categories: git & github
---

`git pull`和`git fetch`作用基本一样，但是用法上区别比较大。
<!-- more -->

> 太长不看版：
>
>       git pull = git fetch + git merge

其实与其说这两个命令的区别，不如说这两个命令的关系。

## git fetch

从远程仓库下载代码到本地。理解`git fetch`，要先理解`FETCH_HEAD`。

`FETCH_HEAD`指的是：`远程仓库中的某个branch的最新状态`。每一个执行过fetch操作的项目，都会存在一个FETCH_HEAD列表，路径是`.git/FETCH_HEAD`文件中，其中的的每一行对应远程仓库的一个分支。每一行的第一列是一个commit_id，它和执行git fetch命令所操作的远程仓库中对应分支的最新commit_id相同。当前分支对应的FETCH_HEAD，就是这个文件的第一行。

执行`git fetch`命令时，分两种情况：

* 没有显式指定fetch哪个远程分支，则FETCH_HEAD默认关联远程的`master`分支
* 如果指定了远程分支，那么FETCH_HEAD就关联这个分支。

常用的`git fetch`使用方式包括一下几种：

> git fetch

这一命令执行两个关键操作：

* 创建并更新所有远程分支的本地分支
* 设定当前分支的`FETCH_HEAD`为远程仓库的master分支（因为没有显式指定远程仓库的分支）

> git fetch origin branch1:branch2

* 执行上面的fetch操作
* 使用远程branch1分支在本地创建branch2分支（但是不会切换到该分支），如果本地不存在branch2分支，则自动创建一个新的branch2分支。如果本地存在branch2分支，并且是'fast forward'，则自动合并两个分支，否则会阻止以上操作。

> git fetch origin branch1

* 设定当前分支的`FETCH_HEAD`为远程服务器的branch1分支。

**注意** 这个操作是`git pull origin branch1`的第一步，而且不会在本地创建新的分支。

`git fetch`更新本地仓库有两种方法：

第一种：

    # 从远程仓库的master分支下载代码到本地的origin/master分支
    $ git fetch origin master

    # 比较本地仓库master分支和origin/master分支的区别（这个origin/master也可以换成FETCH_HEAD）
    $ git log -p master..origin/master 或 git log -p master..FETCH_HEAD

    # 合并远程代码到本地master分支
    $ git merge origin/master 或 git merge FETCH_HEAD

    # 也可以用rebase合并
    $ git rebase origin/master 或 git rebase FETCH_HEAD

第二种：

    # 从远程仓库的master分支下载到本地并创建一个新分支temp
    $ git fetch origin master:temp

    # 比较master分支和temp分支的区别
    $ git diff temp

    # 合并代码
    $ git merge temp 或者 git rebase temp

    # 删除临时分支
    $ git branch -d temp

## git pull

简而言之，`git pull`是`git fetch`加上`git merge FETCH_HEAD`的快捷方式。

当然，如果执行的是`git pull --rebase`，那就是用`git rebase`替换掉`git merge`。

当前的分支情况：

          A---B---C   master on origin
         /
    D---E---F---G master
        ^
        origin/master in your repository

`git pull`后：

          A---B---C   master on origin
         /         \
    D---E---F---G---H master
    
## 小结

`git fetch`是从远程获取最新版本到本地，但是不会自动merge，所以在合并前就可以做一些操作，更安全一些。`git pull`更快捷方便一些。

参考：
[https://ruby-china.org/topics/4768](https://ruby-china.org/topics/4768)

[https://www.jianshu.com/p/e78b28081a44](https://www.jianshu.com/p/e78b28081a44)

[https://stackoverflow.com/questions/9237348/what-does-fetch-head-in-git-mean](https://stackoverflow.com/questions/9237348/what-does-fetch-head-in-git-mean)

[https://segmentfault.com/a/1190000017030384](https://segmentfault.com/a/1190000017030384)