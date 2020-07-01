---
title: 初探babel
date: 2020-02-18 17:02:40
tags: Babel
---

<!-- 初探babel -->

经常做react或者vue项目，会看到项目中`.babelrc`或`babel.config.js`文件，也会看到`package.json`文件中关于babel的依赖。它们是干啥的？看了[这篇文章][1]，有了初步的了解，在此做一下记录。

主要包含一下几个知识点：

* @babel/core，babel的核心模块
* @babel/cli，终端运行工具，允许在终端运行babel
* 插件plugin，本质是一个js程序。babel依靠插件实现代码的转换
* 预设preset，是一组插件的集合。因为实现代码转化可能要用到很多插件，一个个添加太麻烦了~
* 配置，babel通过配置文件中的配置运行。包括`.babelrc`或`babel.config.js`文件
* polyfill，对执行环境的或其他功能的补充。在bable7.4.0后，@babel/polyfill已经不推荐使用，安装`core-js@版本号`，然后在配置文件中配置。
* transform。（这个在文章中没有提及，但也需要学学）

以上这些点先通过文章看一下，然后需要看[官方文档][2]，有助于理解。当然手敲代码是必须的。

[这是一篇babel科普性质的文章](https://zhuanlan.zhihu.com/p/129089156)

[1]: https://juejin.im/post/5e477139f265da574c566dda
[2]: https://www.babeljs.cn/docs/
