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

### 花一点时间观察下属性值的差异

应用更新后，memoizedState和updateQueue中的baseState上的count属性的值更改为1。React也更新了ClickCounter组件实例中的state。

此时，队列中不再有更新，所以firstUpdate为null。重要的是，effectTag属性有了变化。它不再是0，值变为了4。二进制形式是100，这意味着第三个位被设置，这正是表示Update的[side-effect tag](11)的位。

```js
export const Update = 0b00000000100
```

综上所述，当处理父节点ClickCounter Fiber节点时，React调用变化发生前的生命周期方法，更新state并定义相关的side-effect。

## Reconciling children for the ClickCounter Fiber

完成后，React进入[finishClassComponent](12)。在这个方法中，React调用组件实例的rendr方法，并将diffing算法应用到组件返回的子组件上。[文档](13)中介绍了高级概述。这是相关的部分：

```
当比较两个相同类型的React DOM元素时，React查看两者的属性，保持相同的底层DOM节点，只更新有变化的属性。
```

但是，如果我们深入研究，就会发现它实际上是在比较React元素的Fiber节点。但我现在不会讲太多细节，因为这个过程非常复杂。我将写一篇单独的文章，特别关注child reconciliation过程。

如果你想自己学习细节，请查看[reconcileChildrenArray](14)函数，因为在我们的应用程序中，render方法返回一个React元素的数组。

此时，有两件很重要的事要理解。第一，当React进入到child reconciliation 过程中，**它创建或者更新render方法返回的子元素的fiber节点**。finishClassComponent函数返回对当前fiber节点的第一个子节点的引用。它将被赋值给`nextUnitOfWork`，并在之后的work loop中被处理。第二，React将更新子节点的props作为父节点执行work的一部分。为此，它使用从render方法返回的React元素中的数据。

例如，下面是React在协调ClickCounter fiber的子节点之前，span元素对应的Fiber节点的样子：
```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: { children: 0 },
    pendingProps: { children: 0 },
    ...
}
```

如你所见，memoizedProps和pendingProps中的children属性都是0。下面是render方法返回的span元素对应的React元素的结构：
```js
{
    $$typeof: Symbol(react.element)
    key: "2"
    props: { children: 1 }
    ref: null
    type: "span"
}
```

如你所见，在Fiber节点和返回的React元素中的属性存在不同。在用来创建备用Fiber节点的[createWorkInProgress](14)函数中，React将会复制React元素中的更新到Fiber节点中。

所以，React完成ClickCounter组件子节点的协调后，span Fiber节点将会有pendingProps的更新。它们将与span React元素中的值匹配：
```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: { children: 0 },
    pendingProps: { children: 1 },
    ...
}
```

之后，当React执行span Fiber节点的work时，将复制它们到memoizedProps上并添加effect来更新DOM。

这就是React在render阶段为ClickCounter fiber节点执行的所有work。由于button是ClickCounter组件的第一个子元素，它将被赋值给nextUnitOfWork变量。它上面没有事情要做，所以React将移动到它的sibling上，即span Fiber节点。通过[这里描述](15)的算法，这发生在completeUnitOfWork函数中。

## 处理Span fiber的更新

所以，nextUnitOfWork变量现在指向备用的span fiber，React开始在其上work。和执行ClickCounter的步骤相似，我们从[beginWork](8)函数开始。

由于我们的span节点是HostComponent类型，在switch语句中React走这个分支：
```js
function beginWork(current$$1, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        case FunctionalComponent: {...}
        case ClassComponent: {...}
        case HostComponent:
            return updateHostComponent(current, workInProgress, ...)
        case ...
    }
}
```

## reconcile span fiber的子节点

在我们的例子中，在updateHostComponent中span节点上没有发生重要的事。

并结束于[updateHostComponent](16)函数。你可以看到与之类似的class组件调用的updateClassComponent函数，对于functional函数是updateFunctionComponent，等等。你可以在[ReactFiberBeginWork.js](17)文件中找到这些函数。

## 完成span Fiber节点的work

一旦beginWork完成，节点进入completeWork函数。但在这之前，React需要更新span fiber上的memoizedProps。你可能还记得，在协调ClickCounter组件的子组件时，React更新了span Fiber节点上的pendingProps：
```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: {children: 0},
    pendingProps: {children: 1},
    ...
}
```

所以一旦span fiber 的beginWork完成，React将更新pendingProps来匹配memoizedProps:
```js
function performUnitOfWork(workInProgress) {
    ...
    next = beginWork(current$$1, workInProgress, nextRenderExpirationTime)
    workInProgress.memoizedProps = memoizedProps .pendingProps
    ...
}
```

然后，它调用completeWork函数，这个函数和我们之前在beginWork函数看到的相似，基本上是一个大的switch语句：
```js
function completeWork(current, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        case FunctionComponent: {...}
        case ClassComponent: {...}
        case HostComponent: {
            ...
            updateHostComponent(current, workInProgress, ...);
        }
        case ...
    }
}
```

由于我们的span Fiber节点是HostComponent，它运行[updateHostComponent](16)函数。在这个函数中React主要做下面的事情：
 - 准备DOM更新
 - 把更新添加到span fiber的updateQueue中
 - 添加effect来更新DOM

在这些操作被执行之前，span Fiber节点看起来是这样：
```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    effectTag: 0,
    updateQueue: null
    ...
}
```
当work完成后，它看起来是这样：
```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    effectTag: 4,
    updateQueue: ["children", "1"],
    ...
}
```

注意effectTag和updateQueue字段的不同。它不再是0，它的值是4。在二进制中它是100，这表示第三位被设置，这正是Update side-effect tag的位。这是React在接下来的commit阶段为这个节点需要做的唯一的工作。updateQueue字段保存了将用于update的payload。

一旦React处理完ClickCounter和它的子组件，render阶段完成。现在，它可以将已完成的备用树赋值给FiberRoot中的finishedWork属性。这是需要刷新到屏幕上的新树。它可以在render阶段后立即被处理，或者在浏览器给React时间之后进行处理。

## Effects list

在我们的例子中，由于span节点和ClickCounter组件都有side effects，React将向HostFiber的firstEffect属性中添加一个指向span Fiber节点的链接。

React在[compliteUnitOfWork](18)函数中创建effects list。下面是一棵带有effects的Fiber树，effects是更新span节点的文本和调用ClickCounter组件的钩子函数：

![][img2]

下面是带有effects的节点的线性列表：

![][img3]

## Commit阶段

这个阶段从[completeRoot](19)函数开始。在它开始执行之前，它设置FiberRoot上的finishedWork属性为null：
```js
root.finishedWork = null
```

不像render阶段，commit阶段总是同步的，因此它可以安全的更新HostRoot以指示commit work已经开始。

commit阶段是React更新DOM，调用变化发生后生命周期函数componentDidUpdate的地方。为了做到这一点，它遍历在前一个render阶段构建的effects列表，并应用它们。

下面是在render阶段为span和ClickCounter节点定义的effects：
```js
{ type: ClickCounter, effectTag: 5 }
{ type: 'span', effectTag: 4 }
```
ClickCounter的effect tag的值是5或者101（二进制），并且定义了Update work，这个work转换为class组件的componentDidUpdate生命周期方法。还设置了最低有效位，以表示在render阶段这个Fiber节点的所有work已经完成。

span的effect tag的值是4或者100（二进制），并且定义了update work，这个work是原生组件的DOM更新。对于span元素，React将需要更新元素的textContent。

## 应用effects

我们看一下React如何应用那些effects。用于应用effects的函数[commitRoot](20)有3个子函数构成:
```js
function commitRoot(root, finishedWork) {
    commitBeforeMutationLifecycles()
    commitAllHostEffect()
    root.current = finishedWork
    commitAllLifeCycles()
}
```

每个子函数都实现了一个循环，该循环遍历effects列表并检查effects的类型。当发现与函数目的有关的effect时，就应用它。在我们的例子中，它将调用ClickCounter组件上的componentDidUpdate生命周期方法，更新span元素的文本。

第一个函数[commitBeforeMutationLifeCycles](21)查找[Snapshot](22) effect，并调用getSnapshotBeforeUpdate方法。但是，由于我们在ClickCounter组件上没有实现该方法，React在render阶段不会添加这个effect。所以在我们的例子中，这个函数不做任何事。

## DOM 更新

下一步，React移动到[commitAllHostEffects](23)函数。在这个函数中，React将把span元素的文本从0变为1。ClickCounter fiberu需要做任何事情，因为class组件对应的节点没有任何DOM更新。

这个函数的要点是它选择正确类型的effect，并应用于对应的操作。在我们的例子中，我们需要更新span元素的文本，所以我们执行Update分支：
```js
function updateHostEffects() {
    switch (primaryEffectTag) {
        case Placement: {...}
        case PlacementAndUpdate: {...}
        case Update: 
            {
                var current = nextEffect.alternate
                commitWork(current, nextEffect)
                break
            }
        case Deletion: {...}
    }
}
```

通过执行commitWork，我们最终进入到[updateDOMProperties](24)函数。它获取在render阶段添加到Fiber节点的updateQueue负载，并更新span元素上的textConent属性：
```js
function updateDOMProperties(domElement, updatePayload, ...) {
    for (let i = 0; i < updatePayload.length; i += 2) {
        const propKey = updatePayload[i];
        const propValue = updatePayload[i + 1];
        if (propKey === STYLE) {...}
        else if (propKey === DANGEROUSLY_SET_INNER_HTML) {...}
        else if (propKey === CHILDERN) {
            setTextContent(domElement, propValue);
        } else {...}
    }
}
```

DOM更新被应用后，React将finishedWork树赋值给HostRoot。它将备用树设置为当前树：
```js
root.current = finishedWork
```

## 调用变化发生后的生命周期钩子

最后剩下的函数是[commitAllLifecycles](25)。这是React调用变化发生后的生命周期方法的地方。在render阶段，React给ClickCounter组件添加Update effect。这是commitAllLifecycles函数查找的effects之一，并调用componentDidUpdate方法：
```js
function commitAllLifeCyles(finishedRoot, ...) {
    while (nextEffect !== null) {
        const effectTag = nextEffect.effectTag

        if (effectTag & (Update | Callback)) {
            const current = nextEffect.alernate
            commitLifeCycles(finishedRoot, current, nextEffect, ...)
        }

        if (effectTag & Ref) {
            commitAttachRef(nectEffect)
        }

        nextEffect = nextEffect.nextEffect
    }
}
```

这个函数也更新refs，但是由于我们没有此功能，因此不会使用。下面是[commitLifeCycles](26)函数中调用的方法：
```js
function commitLifiCycles(finishedRoot, current, ...) {
    ...
    switch (finishedWork.tag) {
        case FunctionalCompoent: {...}
        case ClassCompoent: {
            const instance = finishedWork.stateNode
            if (finishedWork.effectTag & Update) {
                if (current === null) {
                    instance.componentDidMount()
                } else {
                    instance.componentDidUpdate(prevProps, prevState, ...)
                }
            }
        }
        case HostComponent: {...}
        case ...
    }
}
```

你也可以看到，这个函数是React调用第一次渲染的组件的componentDidMount方法的地方。

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
[11]: https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js
[12]: https://github.com/facebook/react/blob/340bfd9393e8173adca5380e6587e1ea1a23cefa/packages/react-reconciler/src/ReactFiberBeginWork.js#L355
[13]: https://reactjs.org/docs/reconciliation.html#the-diffing-algorithm
[14]: https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L326
[15]: https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e
[16]: https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L686
[17]: https://github.com/facebook/react/blob/1034e26fe5e42ba07492a736da7bdf5bf2108bc6/packages/react-reconciler/src/ReactFiberBeginWork.js
[18]: https://github.com/facebook/react/blob/d5e1bf07d086e4fc1998653331adecddcd0f5274/packages/react-reconciler/src/ReactFiberScheduler.js#L999
[19]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306
[20]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523
[21]: https://github.com/facebook/react/blob/fefa1269e2a67fa5ef0992d5cc1d6114b7948b7e/packages/react-reconciler/src/ReactFiberCommitWork.js#L183
[22]: https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js#L25
[23]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L376
[24]: https://github.com/facebook/react/blob/8a8d973d3cc5623676a84f87af66ef9259c3937c/packages/react-dom/src/client/ReactDOMComponent.js#L326
[25]: https://github.com/facebook/react/blob/d5e1bf07d086e4fc1998653331adecddcd0f5274/packages/react-reconciler/src/ReactFiberScheduler.js#L479
[26]: https://github.com/facebook/react/blob/e58ecda9a2381735f2c326ee99a1ffa6486321ab/packages/react-reconciler/src/ReactFiberCommitWork.js#L351

[img1]: https://admin.indepth.dev/content/images/2019/08/tmp4.gif
[img2]: https://admin.indepth.dev/content/images/2019/08/image.png
[img3]: https://admin.indepth.dev/content/images/2019/08/image-1.png