---
title: 学习React中state和props更新
date: 2020-08-24 21:38:52
tags:
---

了解React中state和props更新的原理

<!-- more -->

# 深入讲解React中state和props更新
[原文：In-depth explanation of state and props update in React](1)

本文使用带有父组件和子组件的基本设置来演示Fiber架构中的内部流程，React依赖该流程将props传递给子组件。

在我的前一篇文章[深入了解React中的新reconciliation算法](2)中，我奠定了理解本文将介绍的更新过程的技术细节所需的基础。

我概述了我将在本中将用到的主要数据结构和概念，特别是Fiber节点，current和work-in-progress树，side-effects和effects列表。我还提供了主要算法的高级概述，解释了render和commit阶段的不同。如果你还没有读它，推荐从那篇文章开始。

我已经向你介绍了这个简单应用，用一个按钮简单的增加渲染在屏幕上的数字：

![][img1]

你可以在[这儿](3)操作.它作为一个简单的组件执行，这个组件render方法返回两个子元素button和span。当你点击button时，组件的状态在处理程序中更新。这导致了span元素文本内容的更新：
```jsx
class ClickCounter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {count: 0};
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState((state) => {
            return {count: state.count + 1};
        });
    }
    
    componentDidUpdate() {}

    render() {
        return [
            <button key="1" onClick={this.handleClick}>Update counter</button>,
            <span key="2">{this.state.count}</span>
        ]
    }
}
```

这里我给组件中增加了componentDidUpdate生命周期方法。这是为了演示在commit阶段，React如何增加effects来调用这个方法。

在本文中，我会向你展示React如何处理state更新和构建effects列表。我们将浏览在render和commit阶段，在高级函数中发生的事情。

特别地，我们将在completeWork函数中，React将如何：

- 更新ClickCounter的state中的count属性
- 调用render方法来获取children列表，执行比较
- 为span元素更新props

以及在commitRoot函数中，React将如何：

- 更新span元素的textContent属性
- 调用componentDidUpdate生命周期方法

不过在那之前，我们将迅速看一下当我们在点击事件处理程序中调用setState时，work是如何调度的。

注意，在使用React时你并不需要知道这些。本文是关于React内部是如何工作的。

## Scheduling updates

当我们点击按钮时，click事件被触发，React执行我们在button的props属性传递的回调函数。在我们的应用中，它只是增加计数器和更新state：
```jsx
class ClickCounter extends React.Component {
    // ...
    handleClick() {
        this.setState(state => {
            return { count: state.count + 1 }
        })
    }
}
```

每个React组件都有一个相关联的updater，充当这个组件和React核心之间的桥梁。这允许ReactDOM，React Native，服务端渲染和测试工具以不同的方式实现setState。

在本文中，我们将会研究updater object在ReactDOM中的实现，ReactDOM中使用了Fiber reconciler。对ClickCounter组件来说它是[classComponentUpdater](4)。它负责检索Fiber实例，排队更新和调度work。

当更新被排队时，它们基本是只是被添加到一个Fiber节点的待处理更新队列中。在我们的例子中，与ClickCounter组件相对应的Fiber节点将会有下面的结构：
```js
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    updateQueue: {
        baseState: { count: 0 },
        firstUpdate: {
            next: {
                payload: (state) => { return {count: state.count + 1 }}
            }
        },
        ...
    },
    ...
}
```

如你所看到的，在`updateQueue.firstUpdate.next.payload`中的函数就是我们在ClickCounter组件中传递给setState的回掉函数。它代表了在render阶段需要被处理的第一个更新。

## 处理ClickCounter Fiber节点的更新

我前一篇文章中的[work loop章节](5)解释了全局变量`nextUnitOfWork`的角色。特别是，它声明这个变量持有对`workInProgress`树中fiber节点的引用，该节点有一些work需要完成。当React遍历Fiber树时，它用这个变量来知道是否有其他带有未完成work的fiber节点。

让我们先假设`setState`方法已经被调用。React把setState的回调函数添加到ClickCounter fiber节点上的`updateQueue`上，并调度工作。React进入render阶段。它使用[renderRoot](6)函数从最顶层的`HostRoot` Fiber节点开始遍历。但是它跳出（忽略）已经处理过的Fiber节点，直到找到一个有未完成work的节点。此时，只有一个Fiber节点需要完成一些work。它是ClickCounter Fiber节点。

所有work都在这个Fiber节点的克隆副本上执行，并存储在`alernate`字段中。如果这个备用节点还没有被创建，在处理更新之前，React在[createWorkInProgress](7)函数中创建副本。我们假设变量`nextUnitOfWork`持有对备用ClickCounter Fiber节点的引用。

## beginWork

首先，我们的Fiber进入[beginWork](8)函数。

由于这个函数会为树中的每个Fiber节点执行，所以如果你想要调试（debug）render阶段，可以在这里放置断点。我经常这样做，并检查一个Fiber节点的类型，以确定我需要的节点。

`beginWOrk`函数基本上是一个很大的`switch`语句，它通过标签来确定一个Fiber节点需要完成的工作的类型。然后执行相应的函数来执行work。在CountClicks的例子中，它是一个class组件，所以采用这个分支：
```js
function beginWork(current$$1, workInProgress, ...){
    switch(workInProgress.tag){
        ...
        case FunctionalComponent: {...}
        case ClassComponent: {
            ...
            return updateClassComponent(current$$1, workInProgress, ...);
        }
        case HostComponent: {...}
        case ...
    }
}
```
然后我们进入[updateClassComponent](9)函数。根据是否是组件的第一渲染，work是否恢复，或者是否是React更新，React要么创建一个实例并挂在组件，要么只更新它：
```js
function updateClassComponent(current, workInProgress, Component, ...) {
    const instance = workInProgress.stateNode;
    let shouldUpdate;
    if (instance === null) {
        ...
        // In the initial pass we might need to construct the instance.
        constructClassInstance(workInProgress, Component, ...)
        mountClassInstance(workInProgress, Component, ...)
        shouldUpdate = true
    } else if (current === null) {
        // In a resume, we'll already have an instance we can resue.
        shouldUpdate = resumeMountClassInstance(workInPregress, Component, ...)
    } else {
        shouldUpdate = updateClassInstance(current, workInProtress, Component, ...)
    }
    return finishClassComponent(current, workInProgress, Component, shouldUpdate, ...)
}
```

## 处理ClickCounter Fiber的更新

我们已经有了ClickCounter组件的实例，所以我们进入[updateClassInstance](10)函数。React在这里执行class组件的大部分work。以下是在函数中执行的最重要的操作，按执行顺序排列：

- 调用`UNSAFE_componentWillReceivePorps()`钩子（已废弃）
- 处理`updateQueue`中的更新，生成新的state
- 使用新state调用`getDerivedStateFromProps`并获得结果
- 调用`shouldComponentUpdate`以确保组件要更新；

    如果为false，则跳过整个渲染过程，包括在这个组件及其子组件上的render调用。否则继续更新。

- 调用`USAFE_componentWillUpdate`（已废弃）
- 增加一个effect来出发`componentDidUpdate`生命周期钩子

    虽然调用`componentDidUpdate`的effect是在render阶段添加，但是这个方法将会在接下来的commit阶段执行。

- 更新组件实例上的state和props

组件实例上更新state和props应该在render方法调用之前，因为render方法的输出通常依赖于state和props。

下面是这个函数的简化版本：
```js
function updateClassInstance(current, workInProgress, ctor, newProps, ...) {
    const instance = workInProgress.stateNode

    const oldProps = workInProgress.memizedPorps
    instance.props = oldProps
    if (oldProps !== newProps) {
        callComponentWillReceivePorps(workInProgress, instance, newPorps, ..)
    }

    let updateQueue = workInProgress.updateQueue
    if (updateQueue !== null) {
        processUpdateQueue(workInProgress, updateQueue, ...);
        newState = workInProgress.memoizedState;
    }

    applyDerivedStateFromPorps(workInProgress, updateQueue, ...)
    newState = workInProgress.memoizedState

    const shouldUpdate = checkShouldComponentUpdate((workInProgress, ctor, ...)
    if (shouldUpdate) {
        instance.componentWillUpdate(newPorps, newState, nextContext)
        workInProgress.effectTag |= Update
        workInProgress.effectTag |= Snapshot
    }

    instance.props = newProps
    instance.state = newState

    return shouldUpdate;
}
```

我已经删除了上面代码段中的一些辅助代码。例如在调用生命周期方法或者添加effects来触发它们之前，React会检查组件是否用`typeof`操作符校验过这个方法。下面例子是React如何在添加effect之前检查componentDidUpdate方法：
```js
if (typeof instance.componentDidUpdate === 'function') {
    workInProgress.effect |= Update
}
```

所以现在我们知道了ClickCounter Fiber节点在render阶段执行了哪些操作。让我们看一下这些操作如何改变Fiber节点的值。当React开始work，ClickCounter组件的Fiber节点看起来像这样：
```js
{
    effectTag: 0,
    elementType: class ClickCounter,
    firstEffect: null,
    memizedState: { count: 0 },
    type: class ClickCounter,
    stateNode: {
        state: { count: 0 }
    },
    updateQueue: {
        baseState: { count: 0 },
        firstUpdate: {
            next: {
                payload: (state, props) => {...}
            }
        }
    },
    ...
}
```

work完成之后，我们最终得到一个如下所示的Fiber节点：
```js
{
    effectTag: 4,
    elementType: class ClickCounter,
    firstEffect: null,
    memizedState: { count: 1 },
    type: class ClickCounter,
    stateNode: {
        state: { count: 1 }
    },
    baseState: { count: 1 },
    firstUpdate: null,
    ...
}
```

[1]: https://indepth.dev/in-depth-explanation-of-state-and-props-update-in-react/
[2]: https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
[3]: https://stackblitz.com/edit/react-jwqn64
[4]: https://github.com/facebook/react/blob/6938dcaacbffb901df27782b7821836961a5b68d/packages/react-reconciler/src/ReactFiberClassComponent.js#L186
[5]: https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
[6]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132
[7]: https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L326
[8]: https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489
[9]: https://github.com/facebook/react/blob/1034e26fe5e42ba07492a736da7bdf5bf2108bc6/packages/react-reconciler/src/ReactFiberBeginWork.js#L428
[10]: https://github.com/facebook/react/blob/6938dcaacbffb901df27782b7821836961a5b68d/packages/react-reconciler/src/ReactFiberClassComponent.js#L976

[img1]: https://admin.indepth.dev/content/images/2019/08/tmp4.gif