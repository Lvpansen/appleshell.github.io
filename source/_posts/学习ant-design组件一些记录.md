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

      flex布局的属性分为容器属性和成员属性，需要分别记忆。
      
      自己模糊的点是成员属性中的简写属性flex：其是flex-grow、flex-shrink、flex-basis三个属性的简写。可通过菜鸟教程学习这三个属性的表现。
      [三个属性学习文章](https://blog.csdn.net/m0_37058714/article/details/80765562)

  4. ant-desing是通过less的函数来计算col成员的flex-basis属性值，采用百分比。

## Avatar 头像

  1. 头像组件中但设置内容为string时，如果字符长度超过了头像容器宽度，设置文本不换行，超出隐藏，然后使用`transform: scale()`样式，缩小内容。缩小的比例是：容器宽度-8 / 字符的宽度。