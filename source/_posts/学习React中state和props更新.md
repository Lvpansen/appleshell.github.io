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

[1]: https://indepth.dev/in-depth-explanation-of-state-and-props-update-in-react/
[2]: https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
[3]: https://stackblitz.com/edit/react-jwqn64

[img1]: https://admin.indepth.dev/content/images/2019/08/tmp4.gif