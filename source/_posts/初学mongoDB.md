---
title: 初学mongoDB
date: 2019-12-22 23:01:31
tags: mongoDB
categories: mongoDB
---

记录初学mongoDB的一些知识点

<!-- more -->

> mongoDB的结构

* 数据库（datebase）
* 集合（collection）
* 文档（document）

它们的关系是一个mongoDB服务中可以包含多个数据库，一个数据库中可以包含多个集合，一个集合中可以包含多个文档。

文档是数据库中的最小单位，我们存储和操作的内容都是文档。

> 操作数据库

    <!-- 显示当前服务下的所有数据库 -->
    show dbs 

    <!-- 显示当前数据库中的所有集合 -->
    show collections

    <!-- 切换数据库 -->
    use <database>
