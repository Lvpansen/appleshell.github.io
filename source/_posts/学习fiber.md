---
title: 学习fiber
date: 2020-08-19 19:49:58
tags: React
---

了解fiber，尝试翻译一篇文章

<!-- more -->

## Inside Fiber: 深入了解React的新协调算法

原文：[Inside Fiber: in-depth overview of the new reconciliation algorithm in React](1)


深入React的新结构：Fiber，了解新协调算法的两个主要阶段。我们将详细介绍React如何更新state和props，以及处理children。

React是一个用于构建用户界面的Javascript库。它的核心[机制](2)是追踪组件状态的变化和把变化的状态投射到屏幕上。React中这个称之为协调（reconciliation）。我们调用`setState`方法，然后框架检查state或者props是否变化，然后重新渲染UI上的组件。

React的官方文档提供了该机制的[高级概述](3)：React元素的角色，生命周期方法，`render`方法，以及应用到子组件的diff算法。`render`方法返回的不可变React元素树，称之为“虚拟DOM”（virtual DOM）。在早期这个术语向人们解释了React，但也带来了的困惑，因此React文档中不再使用它。本文中我将严格地称它为React元素树。

除了React元素树，框架始终有一个内部实例树（组件，DOM节点等），用于保持状态。从React 16版本开始，React推出了该内部实例树的新实现以及管理它的算法，其代号为Fiber。要了解Fiber架构带来的优势，请查看[React在Fiber中使用链表的方式和原因](4)

本文是该系列的第一篇文章，旨在向您介绍React的内部架构。在文中我想提供与算法相关的重要概念和数据结构的深入概述。一旦我们有了足够的背景知识，我们将探索用于遍历和处理fiber树的算法和主要函数。本系列的下一篇文章将展示React如何使用该算法来执行初始化渲染以及处理state和props的更新。从那里我们将继续讨论调度程序，子协调流程以及构建effects list的机制。

在这里我将给你一些非常高级的知识。我鼓励你阅读它以理解Concurrent React内部工作背后的魔法。如果你打算给react贡献代码，这个系列的文章也会给你提供引导。我坚信逆向工程，因此会有很多指向最新版本16.6.0的资源的链接。

要理解的东西肯定很多，所以如果你不能马上理解一些东西时，不要感到有压力。这件事花费时间，但都是值得的。注意，使用React，你并不需要了解这些。本文是关于React内部如何工作的。

### 背景设置



[1]: https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
[2]: https://indepth.dev/what-every-front-end-developer-should-know-about-change-detection-in-angular-and-react/
[3]: https://reactjs.org/docs/reconciliation.html
[4]: https://indepth.dev/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/