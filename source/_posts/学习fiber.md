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

### 设置背景

这是一个我将在整个系列中使用的简单应用。我们有一个按钮，可以简单地增加屏幕上呈现的数字。

![simple application][img1]

下面是实现：

```jsx
class ClickCounter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState((state) => {
      return { count: state.count + 1 }
    })
  }

  render() {
    return [
      <button key="1" onClick={this.handleClick}>Update counter</button>
      <span key="2">{this.state.count}</span>
    ]
  }
}
```

你可以在[这里](5)查看它。正如你所见，它是一个简单的组件，从render方法中返回button和span元素。一旦你单击按钮，组件的状态就在处理程序中更新。这就进而导致span元素的文本更新。

在renconciliaton过程中，React会执行各种活动。例如：以下是在我们简单的应用程序中，React在第一次渲染期间和状态更新后执行的高级操作：

- 更新ClickCounter组件state中的count
- 检索并比较ClickCounter的子元素及其props
- 更新span元素的props

在reconciliation过程中还有其他活动执行，例如调用生命周期函数或者更新refs。所有这些活动在Fiber架构中统称为“work”。work的类型通常依赖于React元素的类型。例如，对于class组件，React需要创建一个实例，但是在函数组件并不会这么做。如你所知，react中有很多种的元素，例如，class和函数组件，原生组件（DOM节点），portals等。React元素的类型由[createElement](6)函数的第一个参数定义。这个函数通常在render方法中用于创建元素。

在开始我们探索活动和主要fiber算法之前，让我们首先熟悉React内部使用的数据结构。

### 从React元素到Fiber节点

React中的每一个组件都有一个UI表示形式，我们可以称之为视图或模板，它们由render方法返回。下面就是我们的ClickCounter组件的模板：
```html
<button key="1" onClick={this.handleClick}>Update counter</button>
<span key="2">{this.state.count}</span>
```

#### React元素

模板通过JSX编译器之后，你将会得到一组React元素。这是React组件render方法真正返回的东西，不是HTML。因为我们可以不使用JSX，ClickCounter组件的render方法重写为下面的形式：
```js
class ClickCounter {
  // ...
  render() {
    return [
      React.createElement(
        'button',
        {
          key: '1',
          onClick: this.onClick
        },
        'Update counter'
      ),
      React.createElement(
        'span',
        {
          key: '2'
        },
        this.state.count
      )
    ]
  }
}
```
render方法中调用React.createElement将会产生下面两种数据结构：
```js
[
  {
    $$typeof: Symbol(react.element),
    type: 'button',
    key: "1",
    props: {
      children: 'Update counter',
      onClick: () => {...}
    }
  },
  {
    $$typeof: Symbol(react.element),
    type: 'span',
    key: "2",
    props: {
      children: 0
    }
  }
]
```
你可以看到React给这些对象增加$$typeof属性，以将他们唯一地标识为React元素。然后我们有了type，key和props属性来描述元素。这些值取自你传给React.createElement函数的值。请注意React如何将文本内容标识为span和button节点的children。以及点击处理函数如何成为button元素props的一部分。React元素上还有其他字段（例如ref字段）不在本位讨论的范围内。

ClickCounter的react元素没有任何props或者key：
```js
{
  $$typeof: Symbol(react.element),
  key: null,
  props: {},
  ref: null,
  type: ClickCounter
}
```

#### Fiber节点

reconciliation过程中，render方法返回的每个React元素的数据被合并到fiber节点树中。不像React元素，fiber节点在每次渲染中不会被重复创建。这些是可变的数据结构，用于保存组件状态和DOM。

我们之前讨论过，根据一个React元素的类型，框架需要执行不同的活动。在我们的简单应用中，对于class组件ClickCounter，它调用生命周期方法和render方法；对于span原生组件（DOM节点）它执行DOM变更。所以每个React元素被转换为[对应的类型](7)的Fiber节点，该节点描述了需要完成的work。

**你可以将fiber视为一种代表某些要完成的工作，或者换句话说，一个工作单元，的数据结构。Fiber的架构也提供了一种方便的方式来追踪，调度，暂停和终止work**

当一个React元素第一次被转化为一个fiber节点时，React使用元素数据在[createFiberFromTypeAndProps](8)函数中创建一个fiber。在后续的更新中，React重用fiber节点，并使用来自相应Rreact元素的数据仅仅更新必要的属性。React也可能需要根据key属性移动层次结构中的节点，或者如果相应React元素不再从render方法中返回，则需要删除它。

检查[ChildReconciler](9)函数，查看React为现有fiber节点执行的所有活动和相应函数的列表。

因为React为每个React元素创建一个fiber，且我们有一棵这些元素的树，因此我们就会有一个fiber节点树。在我们简单应用的例子中长这样：

![fiber tree][img2]

所有fiber节点通过链表连接起来，这个链表使用fiber节点的child，sibling和return属性。更多关于它为什么以这种方式运行，查看[The how and why on React’s usage of linked list in Fiber](10)

#### Current and work in progress trees

第一次渲染后，React得到一棵fiber树，该树反映了用于渲染UI的应用程序的状态。这棵树通常被称为**current**。当React开始更新工作时，它会构建一棵所谓workInProgress树，该树反映将要刷新到屏幕上的未来状态。

所有工作都在workInProgress树的fiber上执行。当React经过current树时，它为每个现存的fiber节点创建一个替代节点，该节点构成了workInProgress树。这个节点使用render方法返回的React元素的数据创建。一旦更新处理完并完成相关工作，React将有一棵准备刷新到屏幕上的替代树。这棵workInProgress树渲染到屏幕上后，它就变成current树。

React的核心原则这一是一致性。React总是一次性更新DOM，不会显示部分结果。workInProgress树用作用户不可见的草稿，因此React可以先处理所有组件，然后将变化刷新到屏幕上。

在源码中你可以看到很多从current和workInProgress树中获取fiber节点作为参数的函数。下面是其中一个函数：
```js
function updateHostComponent(current, workInProgress, renderExpirationTime){...}
```

每个fiber节点在**alternate**字段中保存了其他树中对应节点的引用。一个current树中的节点指向了workInProgress树中的对应节点。

#### side-effects

我们可以将React中的组件视为使用state和props来计算UI表示的函数。其他所有活动，例如更改DOM或者调用生命周期方法，都应该该被视为side-effect,或者简单说，是effect。Effects在[文档](11)中也有提及：

在以前，你可能执行数据获取，订阅，或者手动修改React组件的DOM。我们称这些操作为“side effects”（或者简单说“effects”），因为它们能够影响其他组件而且在渲染过程中不能执行。

你可以看到大多数state和props更新将如何导致side-effect。而且由于应用effects是work的一种，一个fiber节点是一种便捷的机制来追踪effects和更新。每个fiber节点都可以有与其关联的effects。它们被编码在effectTag字段中。

所以Fiber中的effect定义了更新被处理之后实例需要完成的work。对原生组件（DOM元素）来说，work由添加，更新或者删除元素组成。对class组件来说，React可能需要更新refs，调用componentDidMount和componentDidUpdate生命周期函数。还有其他effects对应到其他类型的fiber。

#### Effects list

React处理更新非常快，并且为了达到这个性能水平，它采用了一些有趣的技术。**其中之一就是建立具有effects的fiber节点的线性列表，以实现快速遍历。**遍历线性列表要比遍历一棵树快得多，1而且没有必要花时间处理没有side-effects的节点。

这个列表的目标是创建有DOM更新或者跟其有关联的其他effect的节点。这个列表是finishedWork树的一个子集，使用nextEffect属性链接，而不是在current和workInProgress树中使用的child属性。

Dan Abramov提供了effects列表的类比。他喜欢把它当成一棵圣诞树，用“圣诞灯”把有effect的节点绑在一起。为了形象化，我们想象下面的fiber节点树，高亮的节点表示有work要做。例如我们的更新使c2插入到DOM中，d2和c1改变属性，b2调用生命周期方法。effect列表将会把他们连接在一起，以使React随后能够跳过其他节点。

![effect list][img3]

你能够看到拥有effects的节点是如何连接到一起的。当遍历节点时，React使用firstEffect指针标识列表的起始位置。所以上图可以表示为下图中的线性列表：

![linear list][img4]

#### Root of the fiber tree

每个React应用都有一个或多个DOM元素来作为容器。在我们的例子中id属性为container的div元素。
```js
const domContainer = document.querySelector('#container');
ReactDOM.render(React.createElement(ClickCounter),domContainer)
```

React给每个容器创建一个[fiber root](12)对象，你可以使用对DOM元素的引用来方位它。
```js
const fiberRoot = query('#container')._reactRootContainer._internalRoot
```

这个fiber root是React保存对fiber树的引用的地方。它存储在fiber root的current属性中:
```js
const hostRootFiberNode = fiberRoot.current
```

fiber树从一个特殊的fiber节点类型（即HostRoot）开始。它在内部创建，并充当最顶层组件的父组件。，从HostRoot类型的fiber节点，通过stateNode属性，可以返回到FiberRoot：
```js
fiberRoot.current.stateNode === fibreRoot  // true
```

你可以通过fiber root访问HostRoot类型fiber节点来探索fiber树。或者通过组件实例来得到一个单独的fiber节点，如下：
```js
compInstance._reactInternalFiber
```

#### Fiber node structure

现在我们来看一下为ClickCounter组件创建的fiber节点的结构：
```js
{
  stateNode: new ClickCounter,
  type: ClickCounter,
  alternate: null,
  key: null,
  updateQueue: null,
  memoizedState: { count: 0 },
  pendingProps: {},
  memoizedProps: {},
  tag: 1,
  effectTag: 0,
  nextEffect: null,
}
```
下面是span DOM元素：
```js
{
  stateNode: new HTMLSpanElement,
  type: "span",
  alternate: null,
  key: "2",
  updateQueue: null,
  memoizedState: null,
  pendingProps: { children: 0 },
  memoizedPorps: { children: 0 },
  tag: 5,
  effectTag: 0,
  nextEffect: null
}
```

fiber节点中有很多字段。在前面的内容中我已经描述了alternate，effectTag和nextEffect字段的用途。现在我们看一下为什么还需要其他的。

- stateNode

  保存与fiber节点有关联的组件，DOM节点或者其他React元素类型的类实例的引用。通常，我们称这个属性用来保存和fiber关联的本地state。

- type

  定义和fiber关联的函数或者class。对于class组件，它指向constructor函数，对与DOM元素它特指HTML标签。我用这个字段来理解fiber节点什么元素关联。

- tag

  定义[fiber的类型](7)。它在reconciliation算法中用来决定哪种work需要被完成。正如之前提到的，work的不同取决于React元素的类型。[createFiberFromTypeAndPorps](8)函数将React元素映射到相应的fiber节点类型。在我们的应用中，clickCounter组件的tag属性是1，表示ClassComponent；span元素是5，表示HostComponent。

- updateQueue

  state Updates，callbacks和DOM updates的队列。

- memoizedState

  用于创建输出的fiber的state。处理更新时，它反映了当前屏幕上渲染的state。

- memoizedProps

  前一次渲染中用来创建输出的fiber的props。

- pendingProps

  已经由React元素中的新数据更新，需要被应用到子组件或者DOM元素的props。

- key

  一组子元素的唯一的标识符，用来帮助React计算哪些项发生变化，哪些被添加或者哪些从列表中被删除。它与[此处](13)描述的“列表和key”功能有关。

你可以在[这里](14)找到fiber节点的完整结构。在上面的描述中，我删除了很多字段。特别是我省略了child，sibling和return指针，它们构成了我在[上一篇文章](15)中描述的树形数据结构。还有调用器专用的义类字段，例如expirationTime，childExpirationTime和mode。

### 通用算法

React执行work主要有两个阶段：**render**和**commit**。

在第一次render阶段的过程中React把更新应用到通过setState或者render调度的组件上，计算UI中哪些需要更新，从render方法返回的每个元素，React为其创建一个新的fiber节点。在接下来的更新中，现有react元素的fibers被复用和更新。**这个阶段的结果是一棵用side-effects标记的fiber节点树。**这个effects描述了在接下来的commit阶段中需要被执行的work。在这个阶段中React获取一个被effects标记的的fiber树，然后把他们应用到实例中。它遍历effects列表，执行DOM更新和其他用户可见的改变。

**理解在第一次render阶段中work可以异步执行是很重要的。**React基于可用的时间可以处理一个或多个fiber节点，然后停止并存储已完成的work和让出给某个事件。再然后从它中断的地方继续进行。不过有时候，它可能需要丢弃已完成的work，重新从头开始。在这个阶段执行的work不会导致任何用户可见的变化（比如DOM更新），因此可以实现这些暂停。**相反，在接下来的commit阶段则总是同步的。**这是因为这一步中执行的work会导致用户可见的变化，例如DOM更新。这就是为什么React需要一次完成这些操作。

调用生命周期方法是React执行work的类型之一。一些方法是在render阶段调用，其他则是在commit阶段。下面是在render阶段被调用的生命周期方法：

- [UNSAFE_]componentWillMount (已废弃)
- [UNSAFE_]componentWillReceiveProps (已废弃)
- getDerivedStateFromProps
- shouldComponentUpdate
- [UNSAFE_]componentWillUpdate (已废弃)
- render

如你所见，从16.3版本起，一些在render阶段执行的遗留生命周期方法被标记为UNSAFE。在文档中他们现在被称为遗留的生命周期。他们将会在将来的发布的16.x版本中被弃用，并且没有UNSAFE前缀的副本将会在17.0版本中被移除。

你好奇这样做的原因吗？

好吧，我们刚了解到，因为render阶段不会产生像DOM更新之类的side-effect，所以React可以异步处理更新（升值可以在多线程中执行）。然而，被标记为UNSAFE的生命周期方法经常被误解和巧妙滥用。开发者倾向于把带有side-effect的代码放进这些方法中，这可能会导致新的render方法出现问题。尽管只有没有USAFE前缀的对应项会被移除，他们仍可能在即将出现的Concurrent模式（你可以选择不使用）中引起问题。

下面是在commit阶段执行的生命周期方法：

- getSnampshotBeforeUpdate
- componentDidMount
- componentDidUpdate
- componentWillUnmount

因为这些方式在同步的commit阶段执行，它们可能包含side effects，接触到DOM。

好了，现在我们有了背景知识来了解用于遍历树和执行work的通用算法。我们继续深入。

#### Render阶段

reconciliation算法总是使用[renderRoot](16)函数从最顶层的HostRoot类型fiber结点开始。但是，React会退出（跳过）已处理的fiber节点，知道发现work未完成的节点。例如，如果你在组件树的深处调用setState方法，React将从顶部开始，但是会快速跳过父级直到它到达了调用setState方法的组件。

#### work循环的主要步骤

所有fiber节点都在[work循环](17)中被处理。下面是这个循环同步部分的实现：
```js
function workLoop(isYieldy) {
  if(!isYieldy) {
    while (nextUnitOfWork !== null) {
      nextUnitWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {...}
}
```

在上面的代码中，nextUnitOfWork保存了workInProgress树中fiber节点的引用，该节点有一些work要完成。当React遍历fibers树时，它使用这个变量来知道是否还有其他未完成work的fiber节点。当前fiber被处理完后，这个变量将要么包含树中下一个fiber节点的引用，要么是null。在这种情况下，React将会脱出work循环并准备commit变化。

有4个主要函数用于遍历树，并启动或者完成work：

- [performUnitOfWork](18)
- [beginWork](19)
- [completeUnitOfWork](20)
- [completeWork](21)

为了演示他们是如何使用的，看一下下面的遍历fiber树的动画。我在demo中使用了这些函数的简化实现。每个函数接受一个fiber节点进行处理，当react沿着树向下移动时，你可以看到当前活动的fiber节点发生了变化。在视频中你可以清楚的看到算法是如何从一个分支转到另一个分支的。在移动到父节点之前，它先完成子节点的work。
![step of traverse fiber tree][img5]

注意，笔直的垂直连接表示同级，而弯曲的连接表示子级，例如 b1没有子节点，而b2有一个子节点c1。

[这是视频的链接](22)，你可以暂停播放并检查当前节点和函数的状态。从概念上讲，你可以将“begin”视为“进入”组件，将“complete”视为“退出”。我解释这些函数时，你可以[在此处使用示例和执行](23)

让我们从performUnitOfWork和beginWork这两个函数开始：
```js
function performUnitOfWork(workInProgress) {
  let next = beginWork(workInProgress);
  if (next === null) {
    next = completeUnitOfWork(workInProgress);
  }
  return next
}

function beginWork(workInProgress) {
  console.log('work performed for ' + workInProgress.name);
  return workInProgress.child
}
```
performUnitOfWork函数从workInProgress中接受一个fiber节点，调用beginWork函数开始执行work。这个函数将开始执行一个fiber需要执行的所有活动。为了这个演示的目的，我们简单打印fiber的name来表示已完成额work。**beginWork函数总是返回一个执向在循环中需要处理下一个的子节指针或者null**。

如果下一个子节点存在，它将会在workLoop函数中赋值给变量nextUnitOfWork。但是如果子节点不存在，React知道它抵达了这个分支的末端，所以它会结束当前节点。**一旦这个节点完成了，它将执行sibling节点的work，结束之后返回到父节点。**这个过程在completeUnitOfWork函数执行：
```js
function completeUnitOfWork(workInProgress) {
  while (true) {
    let returnFiber = workInProgress.return;
    let siblingFiber = workInProgress.sibling;

    nextUnitOfWork = completeWork(workInProgress);

    if (siblingFiber !== null) {
      // If there is a sibling, return it to perform work for this sibling
      return siblingFiber;
    } else if (returnFiber !== null) {
      // If there is no more work in this returnFiber,
      // continue the loop to complete the parent
      workInProgress = returnFiber;
      continue;
    } else {
      // we've reache the root
      return null
    }
  }
}

function completeWork(workInProgress) {
  console.log('work completed for ' + workInProgress.name)
  return null
}
```
你可以看到这个函数的核心点是一个大的while循环。当一个workInProgress节点没有子节点时React进入到这个函数。完成当前fiber的work后，它会检查其是否有sibling。如果有，React退出这个函数，返回执行sibling的指针。这个指针赋值给nextUnitOfWork变量，React将从这个sibling开始执行这个分支的work。这里很重要的一点是理解React目前仅完成了之前sibling的work。它还没有完成父节点的work。**只有从子节点开始的所有分支都完成后，它才完成父节点的work然后向上回溯**

正如你从实现中看到的，completeUnitOfWork主要用于迭代目的，而主要的活动发生在beginWork和completeWork函数中。在这个系列接下来的文章中我们将会了解到当React进入到beginWork和completeWork函数中时，ClickCounter组件和span节点发生了什么。

#### commit阶段

这个阶段开始自[completeRoot](24)函数。这是React更新DOM和调用变化发生前后生命周期方法的地方。

当React进入到这一阶段，它有两棵树和effects列表。第一棵树代表当前渲染在屏幕上的状态。然后在render阶段构建一棵备用树。在源码中称之为finishedWork或者workInProgress，代表需要映射到屏幕上的状态。这个备用树通过child和sibling指针类似地链接到当前树。

然后，effect列表--一个finishedWork树的节点子集，通过nextEffect指针连接。记住effect列表是render阶段运行的结果。渲染的全部目的是决定哪些节点需要被插入，更新或者删除，哪些组件需要调用他们的生命周期方法。这就是effect列表告诉我们的。**它正是在commit阶段进行迭代的节点集**

为了debug目的，current树可通过fiber root的current属性访问。finishedWork树可以通过当前树中HostFiber节点的alernate属性访问。

commit阶段运行的主要函数是[commitRoot](25)。基本上，它执行一下操作：

- 在标记为Snapshot effect的节点上调用getSnapshotBeforeUpdate生命周期方法。
- 在标记为Deletion effect的节点上调用componentWillUnmount生命周期方法。
- 执行所有的DOM插入，更新和删除
- 设置finishedWork树为current
- 在标记为Placement effect的节点上调用componentDidMount生命周期方法。
- 在标记为Update effect的节点上调用componentDidUpdate生命周期方法。

调用变化前方法getSnapshotBeforeUpdate后，React commit树中的所有side-effect。这分两步进行。第一步执行所有原生DOM的插入，更新，删除和ref的卸载。然后React把finishedWork树赋值给FiberRoot，把workInProgress树标记为current树。这发生在commit阶段的第一步之后，因此在componentWillUnmount中pervious tree is still current。但是是在第二步之前，因此在componentDidMount/Update中finished work is current。在第二步中，React调用所有其他生命周期方法和ref回调。这些方法作为单独的过程执行，因此整棵树所有的放置（插入），更新，删除操作都已经被调用。

下面是上面描述的函数的执行的核心点：
```js
function commitRoot(root, finishedWork) {
  commitBeforeMutationLifecycles()
  commitAllHostEffects();
  root.current = finishedWork;
  commitAllLifeCycles()
}
```

这些子函数在迭代列表的effect和检查effect类型的循环中执行。当发现与该函数目的相关的effect时，就执行。

#### Pre-mutation lifecycle method

下面是迭代effect树和检查一个节点是否有Snapshot effect的代码：
```js
function commitBeforeMutationLifecycles() {
  while (nextEffect !== null) {
    const effectTag = nextEffect.effectTag;
    if (effectTag & Snapshot) {
      const current = nextEffect.alernate;
      commitBeforeMutationLifecycles(current, nextEffect)
    }
    nextEffect = nextEffect.nectEffect;
  }
}
```

对一个class组件来说，这个effect意味着调用getSnapshotBeforeUpdate生命周期方法。

#### DOM更新

[commitAllHostEffects](26)React执行DOM更新的函数。该函数主要定义了节点需要执行的操作的类型并执行它：
```js
function commitAllHostEffects() {
  let primaryEffectTag = effectTag & (Placement | Update | Deletion);
  switch (primaryEffectTag) {
    case Placement: {
      commitPlacement(nextEffect);
      ...
    }
    case placementAndUpdate: {
      commitPlacement(nextEffect);
      commitWork(current, nextEffect);
      ...
    }
    case Update: {
      commitWork(current, nextEffect);
      ...
    }
    case Deletion: {
      commitDeletaion(nextEffect);
      ...
    }
  }
}
```

有趣的是，React在commitDeletion函数中调用componentWillUnmount方法作为删除过程的一部分。

#### Post-mutation 生命周期fangfa

[commitAllLifecycles](27)是React调用所有剩余生命周期方法componentDidUpdate和componentDidMount的函数。

这个系列的下一篇文章是[深入解释React中的state和props更新](28)

[1]: https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
[2]: https://indepth.dev/what-every-front-end-developer-should-know-about-change-detection-in-angular-and-react/
[3]: https://reactjs.org/docs/reconciliation.html
[4]: https://indepth.dev/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/
[5]: https://stackblitz.com/edit/react-t4rdmh
[6]: https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/react/src/ReactElement.js#L171
[7]: https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/shared/ReactWorkTags.js
[8]: https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L414
[9]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactChildFiber.js#L239
[10]: https://medium.com/dailyjs/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7
[11]: https://reactjs.org/docs/hooks-overview.html#%EF%B8%8F-effect-hook
[12]: https://github.com/facebook/react/blob/0dc0ddc1ef5f90fe48b58f1a1ba753757961fc74/packages/react-reconciler/src/ReactFiberRoot.js#L31
[13]: https://reactjs.org/docs/lists-and-keys.html#keys
[14]: https://github.com/facebook/react/blob/6e4f7c788603dac7fccd227a4852c110b072fe16/packages/react-reconciler/src/ReactFiber.js#L78
[15]: https://indepth.dev/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/
[16]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132
[17]: https://github.com/facebook/react/blob/f765f022534958bcf49120bf23bc1aa665e8f651/packages/react-reconciler/src/ReactFiberScheduler.js#L1136
[18]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1056
[19]: https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489
[20]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L879
[21]: https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532
[23]: https://stackblitz.com/edit/js-ntqfil?file=index.js
[24]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306
[25]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523
[26]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L376
[27]: https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L465
[28]: https://indepth.dev/in-depth-explanation-of-state-and-props-update-in-react/

[img1]: https://admin.indepth.dev/content/images/2019/07/tmp1.gif
[img2]: https://admin.indepth.dev/content/images/2019/07/image-51.png
[img3]: https://admin.indepth.dev/content/images/2019/07/image-52.png
[img4]: https://admin.indepth.dev/content/images/2019/07/image-53.png
[img5]: https://admin.indepth.dev/content/images/2019/08/tmp2.gif