---
title: 学习ant-design组件一些记录
date: 2020-03-21 00:38:12
tags: ant-design react
---

学习ant-design组件的一些笔记。

<!-- more -->

## Grid 栅格

  1. gutter属性是通过context传递给Col组件的。
  2. 在设置gutter属性大于0时，row组件通过设置`margin-left`、`margin-right`，`margin-top`为负值，使得其子组件从视觉上，在左侧，上侧，右侧是与Row组件变源是重合的。

  在实际工作中，如果遇到需要上下左右平均分布，而且边缘和父容器重合的布局时，可以用这种方式事项。
  
  3. 采用的实际是flex布局。