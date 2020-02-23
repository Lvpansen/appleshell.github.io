---
title: github使用小结
date: 2019-12-05 17:55:22
tags: github
categories: git & github
---

主要记录一下github中几个常用的功能和作用，比如，watch、star、fork、topic、找项目、[webhooks][2]等。
<!-- more -->
[watch、star、fork的参考][1]

[fork的项目，同步更新源项目的提交][3]

[github action][4]

[给开源仓库提PR][5]

一些知识点：

> 在github的项目中比较两个分支的差异：

  首先进入github中项目页面，例如vue的项目页面：[https://github.com/vuejs/vue][6], 然后在这个地址后面拼上`/comoare`，[https://github.com/vuejs/vue/comoare][7]。然后就可以选择要比较的分支了。

> 在github中按条件搜索项目，以react为例：

* in: 仓库的名字中包含react关键字的项目：`in:name react`，表示仓库的名称中包含`react`；

  `in:readme react`表示README.md文件中包含`react`的项目。

  `in:description react`表示项目描述中包含`react`的项目。

* stars: 根据star数筛选项目：`react stars:>1000`，表示star数大于1000的所有react相关项目；

* forks: 根据star数筛选项目：`react forks:>1000`，表示fork数大于1000的所有react相关项目；

* language: 根据变成语言搜索，`react language:javascript`

* pushed: 最后一次更新时间，`react pushed:>2019-08-01`，表示最后一个更新时间在2019-08-01之后的项目。

这些条件可以随意组合使用，基本上就能比较准确的定位你想要找的项目了。




[1]: https://www.jianshu.com/p/6c366b53ea41
[2]: https://developer.github.com/webhooks/
[3]: https://blog.csdn.net/qq1332479771/article/details/56087333
[4]: http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html
[5]: https://juejin.im/post/5cefbf956fb9a07ed657b831
[6]: https://github.com/vuejs/vue