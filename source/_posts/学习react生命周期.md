---
title: 学习react生命周期
date: 2019-12-15 23:47:36
tags: react
categories: react
---

初学 react，生命周期是一个重要的知识点，记录生命周期的学习相关。推荐学习[React 生命周期官方文档][1]和[React 生命周期图谱][2]

<!-- more -->

### 生命周期介绍

React 组件的生命周期主要分为四个部分

1. 挂载(Mounting)
2. 更新(Updating)
3. 卸载(Unmounting)
4. 错误处理(Error Handling)

React 在 v16.3 之后，生命周期发生了比较大的变化，先看 v16.3 之前的生命周期：
![v16.3之前的生命周期][pic1]
图中少了`componentDidCatch`，这个生命周期函数是在 v16.0 增加的。

v16.3 之后发生的变化是 deprecate 了一组生命周期函数：

- componentWillMount
- componentWillReceiveProps
- componentWillUpdate

同时，新增了两个生命周期函数：

- static getDerivedStateFromProps
- getSnapshotBeforeUpdate

简而言之，可以认为`getDerivedStateFromProps`函数替代了被 deprecate 的三个生命周期函数。这个新的生命周期函数是个静态函数，所以函数体内不能访问 this，可以认为是个纯函数，输出完全由输入决定。

`getSnapshotBeforeUpdate`函数是在 update 过程中，`render`函数执行后被调用。

变化后的生命周期函数图谱在[这里][2]，可以和变化前的对比一下，有助于增进理解。但是这个函数大部分情况下可能用不到。

下面简单介绍下组件在不同阶段的生命周期调用顺序，[官网介绍][1]更详细：

> Mounting

组件实例被创建并插入 DOM 时，调用顺序如下：

- `constructor()`
- `static getDerivedStateFromProps()`
- `render()`
- `componentDidMount()`

> Updating

组件的 props 或 state 发生变化时触发更新，调用顺序如下：

- `static getDerivedStateFromProps()`
- `shouldComponentUpdate()`
- `render()`
- `getSnapshotBeforeUpdate()`
- `componentDidUpdate()`

> Unmounting

组件从 DOM 中移除时，调用如下：

- `componentWillUnmount()`

> Error Handling

组件渲染过程中，当生命周期函数或者子组件构造函数中抛出错误，调用如下方法

- `static getDerivedStateFromError()`
- `componentDidCatch()`

### 小结

1. 为什么 v16.3 会 deprecate 一部分生命周期函数，原因可以参考[这篇文章][3]，介绍的很详细。
2. 多看看[官方文档][1]，尤其是文档中的[生命周期图谱][2]，图谱中可以选择查看常用生命周期函数和不常用生命周期函数。对于理解 React 生命周期有很大帮助。

[1]: https://zh-hans.reactjs.org/docs/react-component.html#the-component-lifecycle
[2]: http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
[3]: https://zhuanlan.zhihu.com/p/38030418
[pic1]: https://pic3.zhimg.com/v2-48e4dd255a7690beaef4d496ac6af7ca_r.jpg
