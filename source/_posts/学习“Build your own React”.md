---
# layout: 学习
title: 学习“Build your own React”
date: 2020-09-06 23:59:40
tags: React
---

学习[Build your own React](1)文章，理解React工作原理。

<!-- more -->

我们将从头开始重写React。一步一步。遵循真实的React代码的架构，但是没有所有的优化和非必要的功能。

如果你读过我以前的“构建自己的React”文章，不同之处在于本文是基于React 16.8，所以我们现在可以使用hooks，并删除所有与class相关的代码。

你可以在旧博客文章找到历史记录，并在Didact仓库中找到代码。还有一个演讲涉及相同的内容。但这是一篇自成体系的文章。

从头开始，这些都是我们将逐一添加到React版本中的所有内容：

- StepⅠ：The `createElement` Function
- StepⅡ：The `render` Function
- StepⅢ：Concurrent Mode
- StepⅣ：Fibers
- StepⅤ：Render and Commit Phases
- StepⅥ：Reconciliation
- StepⅦ：Function Components
- StepⅧ：Hooks

## Step 0：回顾

但是首先让我们回顾一些基本概念。如果你已经很好的掌握了React，JSX和DOM元素如何工作的，可以跳过这一步骤。

我们将使用这个React应用，只有三行代码。第一行定义了一个React元素。第二行获取了DOM中的一个节点。最后一行把React元素渲染到容器中。
```jsx
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDom.render(element, container)
```

让我们删除所有React特定的代码，然偶将其替换为原始的JavaScript。

第一行我们有用JSX定义的元素。它甚至不是有效的JavaScript，所以为了用原始的JavaScript替换它，我们首先需要用有效的JS替换它。

JSX通过像Babel的构建工具转换为JS。这个转换通常很简单：调用createEelment来替换标签内的代码，将标签名称，props和子元素作为参数传递给函数。
```js
const element = createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
```

`React.createElement`根据其参数创建一个对象。除了一些验证，这就是它做的一切。所以我们可以安全地将函数调用替换为它的输出。
```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello"
  }
}
```

这就是元素的样子，包含两个属性的对象：type和props（实际上还有其他属性，但是我们只关心这两个）。

type属性是个字符串，表示我们想要创建的DOM节点的类型，当你想创建一个HTML元素时，它是你传递给document.createElement的tagName。它也可以是个函数，但是我们在Step Ⅶ讲解。

props是另一个对象，它包含了JSX属性上所有的keys和values。它还包含一个特殊的属性：children。

在这个例子中children是一个字符串，但它通常是一个包含多个元素的数组。这就是为什么元素也是树的原因。

另一段我们需要替换的React代码是ReactDOM.render的调用。
```jsx
ReactDOM.render(element, container)
```

render是React改变DOM的地方，所以让我们自己来做更新。

首先我们用元素类型（本例中是h1）创建一个节点。然后我们把元素的所有属性分配给这个节点。本例中只有title。

然后我们为子元素创建节点。我们只有一个字符串作为子元素，所以我们创建一个文本节点。

最后，我们把textNode添加到h1，把h1添加到container
```js
const node = document.createElement(element.type)
node["title"] = element.props.title

const text = document.createElement("")
text["nodeValue"] = element.props.children

node.appendChild(text)
container.appendChild(node)
```

现在我们有一个和之前相同的应用，但是没有用React。

为了避免混淆，我将使用"element"指代React元素，"node"指代DOM元素。

## Step Ⅰ：`createElement`函数

让我们从另一个app开始，这次我们将用我们自己版本的React替换React代码。
```jsx
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```
正如我们在上一步骤中看到的，一个元素是一个包含type和props的对象。我们的函数需要做的唯一的事情是创建那个对象。
```js
const element = React.createElement(
  "div",
  {id: "foo"},
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
```

我们在props上使用扩展操作，在children上使用剩余参数语法，用这种方式，children属性将总是一个数组
```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children,
    }
  }
}
```
例如，`createElement("div")`返回：
```js
{
  type: "div",
  props: {"children": []}
}
```
`createElement("div", null, a)`返回：
```js
{
  type: "div",
  props: {children: [a]}
}
```
`createElement("div", null, a, b)`返回：
```js
{
  type: "div",
  props: {children: [a, b]}
}
```

children数组也可以包含基本值，例如strings和numbers。所以我们将把所有不是对象的内容包装到它自己的元素中，并他们创建特殊的类型：TEXT_ELEMENT
```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === 'object'
          ? child
          : createTextElement(child)
      )
    }
  }
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: [],
    }
  }
}
```
当没有children时，React不会包装基本值或者创建空数组，但是我们这么做是因为它将会简化我们的代码，对于我们的库，我们更喜欢简单代码，而不是性能好的代码。

我们仍然使用React的createElement。为了替换它，我们给我们的库起一个名字。我们需要一个听起来想React但是也暗示了它教学目的的名字。

我们称之为Didact。
```js
const Didact = {
  createElement
}

const element = Didact.createElement(
  "div",
  {id: "foo"},
  Didact.createElement("a", null, "bar"),
  Didact.createElement("b"),
)
```

但是我们在这里仍然想用JSX。我们应该如何告诉babel使用Didact的createElement，替换React的。
```js
/** @jsx Didact.createElment */
const element = (
  <div id="foo">
    <a>bar</a>
    <b/>
  </div>
)
```

如果我们像上面这样注释，当babel转译JSX时，它将使用我们定义的函数。

## Step Ⅱ： render函数

下一步，我们需要写出我们版本中的ReactDOM.render函数。

目前，我们只关心向DOM中添加内容。我们叫稍后处理更新和删除。

我们从使用元素类型创建DOM节点开始，然后把新节点添加到容器中。我们递归地给每个子节点执行相同的操作。我们也需要处理文本节点，如果类型是TEXT_ELEMENT，我们就创建一个文本节点。最后，我们需要把元素的属性赋值给节点。

```js
function render (element, container) {
  const dom =
    element.type === 'TEXT_ELEMENT'
      ? document.createTextNode('')
      : document.createElement(element.type)

  const isProperty = key => key !== 'children'
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })

  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```

就是这样，我们现在有一个将JSX渲染到DOM上的库。

## Step Ⅲ：Comcurrent Mode

但是，在我们开始添加更多代码前，我们需要重构。

这个递归调用有一个问题。

一旦我们开始渲染，在我们渲染完整棵树之前，我们不同停止。如果elelment tree非常大，它可能阻塞主线程非常长的时间。而且，如果浏览器需要处理高优先级的内容，例如处理用户输入或者保证动画流畅，它将不得不等到渲染结束。

所以我们将把work分成小的单元，在完成每个单元后，如果需要执行其他任何操作，我们将让浏览器中断渲染。

```js
let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitWork(nextUnitOfWork)
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}

requestIdleCallback(workLoop)

functino performUnitWork(nextUnitOfWork) {
  // TODO
}
```

我们使用`requestIdleCallback`来进行循环。你可以把`requestIdleCallback`当成是`setTimeout`，但是我们不是告诉它什么时候运行，而是当主线程空闲时运行回调。

React[不再使用requestIdleCallback](2)。现在它使用[scheduler package](3)。但是对于此用例，它在概念上是相同的。

`requestIdleCallback`也给了我们一个deadline参数。我们可以用它来检查在浏览器再次控制之前我们还有多少时间。

截止到2019年11月，Concurrent Mode在React中仍然不稳定。稳定版本的循环看起来更像这样：

```js
while (nextUnitOfWork) {
  nextUnitOfWork = performUnitOfWork(
    nextUnitOfWork
  )
}
```

要开始使用循环，我们需要设置第一个work单元，然后写`performUnitOfWork`函数，这个函数不仅执行work，而且返回下一个work单元。

## Step Ⅳ：Fibers

为了组织work单元，我们需要一个数据结构：fiber tree。

**每个元素都有一个fiber，每个fiber是一个work单元**

让我举一个例子，假如我们想渲染下面这样一棵元素树：
```js
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
  </div>,
  container
)
```

![fiber tree][img1]

在`render`中我们将创建root fiber，并把它设置为`nextUnitOfWork`。剩余的work将在`performUnitOfWork`中进行，我们将在那里做三件事：

  1. 把元素添加到DOM
  2. 为元素的子元素创建fibers
  3. 选择下一个work单元（next unit of work）

这个数据结构的目标之一是使得找到下一个work单元变得容易。这就是为什么每一个fiber都与它的第一个子节点，它的下一个兄弟节点和它的父节点相连。

当我们完成一个fiber上的work时，如果它有子fiber（child），那么该fiber将成为下一个work单元。在我们的例子中，当我们完成div fiber上的work时，下一个work单元将是h1 fiber。

如果fiber没有子fiber（child），我们将用兄弟fiber（sibling）作为下一个work单元。例子中，p fiber没有子fiber，所以完成其上的work后，我们将移动到a fiber上。

如果一个fiber既没有子fiber，也没有兄弟fiber，我们将进入“叔叔”fiber: 父节点的兄弟节点。例子中像a fiber和h2 fiber的关系。

同样，如果父fiber（parent）没有兄弟fiber（sibling），我们就不断向上检查父fiber，直到找到有兄弟节点的父fiber或者到达root。如果我们到达了根，就意味着我们完成了这次render中执行的所有work。

[1]: https://pomb.us/build-your-own-react/
[2]: https://github.com/facebook/react/issues/11171#issuecomment-417349573
[3]: https://github.com/facebook/react/tree/master/packages/scheduler

[img1]: https://pomb.us/static/a88a3ec01855349c14302f6da28e2b0c/ac667/fiber1.png
